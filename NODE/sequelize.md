참고 https://inpa.tistory.com/entry/EXPRESS-%F0%9F%93%9A-%EC%8B%9C%ED%80%84%EB%9D%BC%EC%9D%B4%EC%A6%88-ORM

### Sequelize란?

시퀄라이즈는 nodejs에서 데이터베이스를 쉽게 다룰 수 있도록 도와주는 라이브러리로,
ORM(Object-relational Mapping)으로 분류된다.
sql 작성법을 모르더라도 데이터베이스 관리가 가능하다.

> Tip
> ORM이란 객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결)해주는 것을 말한다.
> 객체 지향 프로그래밍은 클래스를 사용하고, 관계형 데이터베이스는 테이블을 사용한다.

시퀄라이즈는 MySQL 외에도 Mariadb, PostgreSQL 등등 다른 데이터베이스에도 쓸 수 있다.
문법이 어느 정도 호환되므로 프로젝트를 다른 SQL 데이터베이스로 전활할 때도 편리하다.

시퀄라이즈는 자바스크립트 구문을 알아서 SQL로 바꿔준다.
그래서 자바스크립트만으로 MySQL을 조작할 수 있어, SQL 언어를 몰라도 MySQL을 어느 정도 다룰 수 있다는 장점이 있다.

### 시퀄라이즈 프로젝트 구성

시퀄라이즈에 필요한 morgan, sequelize, sequelize-cli, mysql2 패키지를 설치한다.

```
$ npm i express morgan sequelize sequelize-cli mysql2
```

패키지를 설치했다면 시퀄라이즈를 init하여 시퀄라이즈 구조를 생성한다.

```
$ npx sequelize init
```

명령 실행 이후에 폴더트리를 다시 보면 config, models, migrations, seeders 폴더가 생성된 것을 볼 수 있다.
추가적으로 우리가 프로젝트에 쓰일 폴더들도 직접 만들어주면 좋다. public, routes, views, app.js 등등

models 폴더 내의 index.js가 생성되었는지 확인한다.
sequelize-cli가 자동으로 생성해주는 코드는 그대로 사용할 떄 에러가 발생하고, 필요없는 부분도 많으므로 다음과 같이 수정해준다.

#### models/index.js

```javascript
const Sequelize = require("sequelize");
const env = process.env.NODE_ENV || "development"; // 지정된 환경변수가 없으면 'development'로 지정
// config/config.json 파일에 있는 설정값들을 불러온다.
// config객체의 env변수(development)키 의 객체값들을 불러온다.
// 즉, 데이터베이스 설정을 불러온다고 말할 수 있다.
const config = require("../config/config.json")[env];
const db = {};
// new Sequelize를 통해 MySQL 연결 객체를 생성한다.
const sequelize = new Sequelize(
  config.database,
  config.username,
  config.password,
  config
);
// 연결객체를 나중에 재사용하기 위해 db.sequelize에 넣어둔다.
db.sequelize = sequelize;
// 모듈로 꺼낸다.
module.exports = db;
```

#### config/config.json

```json
{
  "development": {
    "username": "root",
    "password": "password",
    "database": "database",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

위 코드의 models/index.js 파일에서 env변수에 process.env.NODE_ENV 또는 'development'로 설정해 주었다.
process.env.NODE_ENV 환경변수를 따로 지정하지 않는 한 변수 env에 기본적으로 'development'가 오기 때문에, development의 환경 설정을 가져오게 된다.
추후 배포할 떄는 process.env.NODE_ENV를 production으로 설정하면 된다.

### 시퀄라이즈 MySQL 연결

#### app.js

```javascript
const express = require("express");
const path = require("path");
const morgan = require("morgan");
// index.js에 있는 db.sequelize 객체 모듈을 구조분해로 불러온다.
const { sequelize } = require("./models");
const app = express();
app.set("port", process.env.PORT || 3000);
// PUG 설정
app.set("views", path.join(__dirname, "views"));
app.set("view engine", "pug");
sequelize
  .sync({ force: false })
  .then(() => {
    console.log("데이터베이스 연결됨.");
  })
  .catch((err) => {
    console.error(err);
  });
app.use(morgan("dev")); // 로그
app.use(express.static(path.join(__dirname, "public"))); // 요청시 기본 경로 설정
app.use(express.json()); // json 파싱
app.use(express.urlencoded({ extended: false })); // uri 파싱
// 일부러 에러 발생시키기 TEST용
app.use((req, res, next) => {
  const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
  error.status = 404;
  next(error);
});
// 에러 처리 미들웨어
app.use((err, req, res, next) => {
  // 템플릿 변수 설정
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== "production" ? err : {}; // 배포용이 아니라면 err설정 아니면 빈 객체
  res.status(err.status || 500);
  res.render("error"); // 템플릿 엔진을 렌더링 하여 응답
});
// 서버 실행
app.listen(app.get("port"), () => {
  console.log(app.get("port"), "번 포트에서 대기 중");
});
```

#### [sequelize.sync() 옵션]

//\* DB 연결 및 생성
sequelize
//? force: true는 모델을 수정하면, 이를 db에 반영하기 위한 옵션이다.
//? 단, 테이블을 지웠다 다시 생성하는 거라서 기존 데이터가 날아간다.
//? 따라서, alter: true 옵션을 통해 기존 데이터를 유지하면서 테이블을 업데이트 할 수 있다.
//? 다만, 필드를 새로 추가할 때 필드가 notnull이면 제약조건에따라 오류가 나는 등 대처를 해야 한다.
.sync({ force: false })
.then(() => {
console.log('데이터 베이스 연결 성공');
})
.catch(err => {
console.error(err);
});

### MySQL 테이블 생성 구문

```javascript
create schema `nodejs` default character set utf8;
use nodejs;
drop table if exists comments;
drop table if exists users;
create table nodejs.users (
id int not null primary key auto_increment,
name varchar(20) not null,
age smallint unsigned not null,
married tinyint not null, -- tinyint는 0과 1 불리언 용
comment text null, -- 자기 소개
created_at datetime not null default now(),
unique index name_unique (name asc)
)
comment = "사용자 정보"
default character set = utf8
engine = InnoDB;
create table nodejs.comments (
id int not null primary key auto_increment,
commenter int not null,
comment varchar(100) not null, -- 댓글
created_at datetime not null default now(),
index commenter_idx(commenter ASC),
constraint commenter foreign key(commenter) references nodejs.users(id) on delete cascade on update cascade
)
comment = "댓글"
default charset = utf8mb4 -- mb4는 이모티콘도 넣을 수 있음
engine = InnoDB;
insert into users(name, age, married, comment) values('zero', 24, 0, '자기소개1');
insert into users(name, age, married, comment) values('nero', 32, 1, '자기소개2');
```

### Sequelize 모델 정의하기

MYSQL의 테이블은 시퀄라이즈의 모델과 대응된다.
시퀄라이즈는 모델과 MySQL의 테이블을 연결해주는 역할을 한다.

시퀄라이즈는 기본적으로 모델 이름은 단수형 (User), 테이블 이름은 (users)으로 사용한다.

#### models/user.js

```javascript
const Sequelize = require("sequelize");
class User extends Sequelize.Model {
  // 스태틱 메소드
  // 테이블에 대한 설정
  static init(sequelize) {
    return super.init(
      {
        // 첫번째 객체 인수는 테이블 필드에 대한 설정
        name: {
          type: Sequelize.STRING(20),
          allowNull: false,
          unique: true,
        },
        age: {
          type: Sequelize.SMALLINT,
          allowNull: false,
        },
        married: {
          type: Sequelize.BOOLEAN,
          allowNull: false,
        },
        comment: {
          type: Sequelize.TEXT,
          allowNull: true,
        },
        created_at: {
          type: Sequelize.DATE,
          allowNull: false,
          defaultValue: Sequelize.NOW,
        },
      },
      {
        // 두번째 객체 인수는 테이블 자체에 대한 설정
        sequelize /* static init 메서드의 매개변수와 연결되는 옵션으로, db.sequelize 객체를 넣어야 한다. */,
        timestamps: false /* true : 각각 레코드가 생성, 수정될 때의 시간이 자동으로 입력된다. */,
        underscored: false /* 카멜 표기법을 스네이크 표기법으로 바꾸는 옵션 */,
        modelName: "User" /* 모델 이름을 설정. */,
        tableName: "users" /* 데이터베이스의 테이블 이름. */,
        paranoid: false /* true : deletedAt이라는 컬럼이 생기고 지운 시각이 기록된다. */,
        charset: "utf8" /* 인코딩 */,
        collate: "utf8_general_ci",
      }
    );
  }
  // 다른 모델과의 관계
  static associate(db) {
    // 인자로 index.js에서 만든 여러 테이블이 저장되어있는 db객체를 받을 것이다.
    db.User.hasMany(db.Comment, {
      foreignKey: "commenter",
      sourceKey: "id",
      onDelete: "cascade",
      onUpdate: "cascade",
    });
    // db.User (hasMany) db.Comment = 1:N 관계 이다.
    // db.User는 가지고있다. 많이. db.Comment를
  }
}
module.exports = User;
```

#### models/comment.js

```javascript
const Sequelize = require("sequelize");
class Comment extends Sequelize.Model {
  static init(sequelize) {
    return super.init(
      {
        comment: {
          type: Sequelize.STRING(100),
          allowNull: false,
        },
        created_at: {
          type: Sequelize.DATE,
          allowNull: true,
          defaultValue: Sequelize.NOW,
        },
      },
      {
        sequelize,
        timestamps: false,
        modelName: "Comment",
        tableName: "comments",
        paranoid: false,
        charset: "utf8mb4",
        collate: "utf8mb4_general_ci",
      }
    );
  }
  static associate(db) {
    db.Comment.belongsTo(db.User, {
      foreignKey: "commenter",
      targetKey: "id",
      onDelete: "cascade",
      onUpdate: "cascade",
    });
    // db.Comment (belongTo) db.User = N:1 관계 이다.
    // db.Comment는 속해있다. db.User에게
  }
}
module.exports = Comment;
```

모델은 Sequelize.Model을 확장한 클래스로 선언한다.

모델은 static init 메서드와 static associate 메서드로 나뉘는데,
init 메서드에서는 테이블에 대한 설정을 하고,
associate 메서드에는 다른 모델과의 관계(1:1, 1:N)를 적는다.

init 메서드의 부모 콜백 super.init 메서드는
첫 번째 인수는 테이블 컬럼에 대한 설정이고,
두 번째 인수는 테이블 자체에 대한 설정이다.

시퀄라이즈는 알아서 id를 기본 키로 연결하므로, id 컬럼은 따로 적어줄 필요는 없다.

### 시퀄라이즈의 자료형

시퀄라이즈의 자료형은 MySQL과 조금 다르다
아래 비교를 통해 맞는 옵션을 입력하자.
![](https://velog.velcdn.com/images/sanizzang00/post/f60462e3-5f7b-4eac-96ef-79357538c32c/image.png)

### 시퀄라이즈 옵션

super.init의 두 번째 인수는 테이블 옵션이다.

- sequelize:
  static init 메서드의 매개변수와 연결되는 옵션으로, db.sequelize 객체를 넣어야 한다.
  나중에 model/index.js에서 연결한다.
- timestamps:
  이 속성 값이 true면 시퀄라이즈는 createdAt과 updatedAt 컬럼을 추가하며, 각각 로우가 생성될 때와 수정될 때의 시간이 자동으로 입력된다.
  (그러나 예제에선 직접 created_at 컬럼을 만들었으므로 지금은 timestamps 속성이 필요없다. 나중엔 꼭 true로 하고 하자)
- underscored:
  시퀄라이즈는 기본적으로 테이블명과 컬럼명을 카멜 표기법 (camel case) 으로 만든다.
  이를 스네이크 표기법 (snake case) 으로 바꾸는 옵션이다 (예를들어 updatedAt 을 updated_at 으로).
- modelName:
  모델 이름을 설정할 수 있다. 노드 프로젝트에서 사용한다.
- tableName:
  실제 데이터베이스의 테이블 이름.
  기본적으로 모델 이름의 소문자 및 복수형으로 만든다.
  예를 들어 모델 이름이 User 라면 테이블 이름은 users 이다.
- paranoid:
  true로 설정하면 deletedAt이라는 컬럼이 생긴다.
  로우를 삭제할 때 완전히 지우지 않고, deletedAt에 지운 시각이 기록된다.
  로우를 조회하는 명령을 내렸을 경우 deletedAt의 값이 null인 로우를 조회한다.
  이렇게 하는 이유는 후에 로우를 복원하기 위해서다. 로우를 복원해야 할 상황이 생길 것 같다면 미리 true로 설정해두자.
- charset / collate:
  각각 utf8 과 utf8_general_ci 로 설정해야 한글이 입력된다.
  이모티콘까지 입력할 수 있게 하고 싶다면 utf8mb4 와 utf8mb4_general_ci 를 입력한다.

#### models/index.js

models/index.js에 두 테이블 파일을 연결시켜준다.

```javascript
const Sequelize = require("sequelize");
// 클래스를 불러온다.
const User = require("./user");
const Comment = require("./comment");
const env = process.env.NODE_ENV || "development";
// config/config.json 파일에 있는 설정값들을 불러온다.
// config객체의 env변수(development)키 의 객체값들을 불러온다.
// 즉, 데이터베이스 설정을 불러온다고 말할 수 있다.
const config = require("../config/config.json")[env];
const db = {};
// new Sequelize를 통해 MySQL 연결 객체를 생성한다.
const sequelize = new Sequelize(
  config.database,
  config.username,
  config.password,
  config
);
// 연결객체를 나중에 재사용하기 위해 db.sequelize에 넣어둔다.
db.sequelize = sequelize;
// 모델 클래스를 넣음.
db.User = User;
db.Comment = Comment;
// 모델과 테이블 종합적인 연결이 설정된다.
User.init(sequelize);
Comment.init(sequelize);
// db객체 안에 있는 모델들 간의 관계가 설정된다.
User.associate(db);
Comment.associate(db);
// 모듈로 꺼낸다.
module.exports = db;
```

db라는 객체에 User와 Comment 모델을 담았다.
앞으로 db 객체를 require하여 User와 Comment 모델에 접근할 수 있을 것이다.
또한 User.init과 Comment.init은 각각의 모델의 static.init 메서드를 호출하며,
init이 실행되어야 테이블이 모델로 연결된다.
다른 테이블과의 관계를 연결하는 associate 메서드 역시 실행해둔다.

### Sequelize 관계 정의하기

#### 1:N 관계 (hasMany, belongsTo)

시퀄라이즈에서는 1:N 관계를 hasMany 메서드로 표현한다.
users 테이블의 로우 하나를 불러올 때 연결된 comments 테이블의 로우들도 같이 불러올 수 있다.

반대로 belongsTo 메서드도 있다.
이는 comments 테이블의 로우를 불러올 때 연결된 users 테이블의 로우를 가져온다.
보통 외래키가 붙은 테이블이 belongTo가 된다.

모델 각각의 static associate 메서드에 정의

```javascript
// user.js
static associate(db) {
	db.User.hasMany(db.Comment, { foreignKey: 'commenter', sourceKey: 'id', onDelete: 'cascade', onUpdate: 'cascade'});
  // db.User (hasMany) db.Comment = 1:N 관계 이다.
  // 남 (db.Comment)의 컬럼 commenter가 내 (db.User) 컬럼 id를 참조하고 있다.
}
```

```javascript
// comment.js
static associate(db) {
    db.Comment.belongsTo(db.User, { foreignKey: 'commenter', targetKey: 'id', onDelete: 'cascade', onUpdate: 'cascade'});
    // db.Comment (belongTo) db.User = N:1 관계 이다.
    // 내(db.Comment)의 컬럼 commenter는 남(db.User) 컬럼 id에 속해 있다.
  }
};
```

간단히 User가 많은 댓글을 가질 수 있으니 User.hasMany가 되는 것이고,
Comment는 한 User에 속할 수 있으니 Comment.belongsTo가 되는 것이다.

둘이 소통하는 키는 foreignKey인 commenter이며,
User의 sourceKey는 곧 Commenter의 targetKey가 된다 (hasMany에서는 sourceKey, belongsTo에서는 targetKey).

foreignKey를 따로 지정하지 않는다면 이름이 모델명+기본 키인 컬럼이 모델에 생성된다.
즉, 예를 들어 위 예제에서 commenter를 foreignKey로 설정해 주지 않았다면, 제약조건에 따라 모델명인 User과 기본 키인 id가 합쳐진 UserId가 foreignKey로 따로 생성된다.

### 1:1 관계(hasOne, belongsTo)

1:1관계에서는 hasMany 대신 hasOne을 사용한다.
foriegnKey, sourceKey, targetKey의 사용법은 1:N 관계와 같다.

hasOne도 1:1, belongsTo도 1:1이면, 누가 기준으로 될지 애매할 때가 있다.
이때 외래키가 붙은 모델은 belongsTo로 기준을 잡아주면 된다.

```javascript
db.User.hasOne(db.Info, { foreignKey: "UserId", sourceKey: "id" });
db.Info.belongsTo(db.User, { foreignKey: "UserId", targetKey: "id" });
```

### N:M 관계 (belongsToMany)

시퀄라이즈에는 N:M 관계를 belongsToMany 메서드로 표현한다.
이 경우엔 어느 한 테이블이 어느 다른 테이블에 종속되는 관계가 아니다.

예를 들어 Post 모델과 Hashtag 모델이 있다고 할 때, 다음과 같이 표현할 수 있다.

```javascript
// Post
db.Post.belongsToMany(db.Hashtag, { through: "PostHashtag" });

// Hashtag
db.Hashtag.belongsToMany(db.Post, { through: "PostHashtag" });
```

N:M 관계의 특성상 새로운 모델이 다음과 같이 생성되며, through 속성에 그 이름을 적으면 된다. 새로 생성된 PostHashtag 모델에는 게시글과 해시태그의 아이디가 저장된다.
N:M에서는 데이터를 조회할 때 여러 단계를 거쳐야 한다.
예를 들어 #노드 해시태그를 사용한 게시물을 조회하는 경우, 먼저 #노드 해시태그를 Hashtag 모델에서 조회하고, 가져온 태그의 아이디인 '1'을 바탕으로 PostHashtag 모델에서 hashtagId가 1인 postId들을 찾아 Post 모델에서 해당 정보를 가져와야 한다.

---

### 시퀄라이즈 쿼리문

CRUD 작업을 하기 위해선 먼저 시퀄라이즈 쿼리를 알아야한다.
시퀄라이즈 쿼리문은 비동기로 동작하며 프로미스 객체를 반환하므로, then을 붙여 결과값을 받을 수 있다.

### 테이블 조회

#### findAll

- 쿼리 결과를 배열 객체로 반환
  모든 데이터를 조회하고 싶으면 findAll 메서드를 사용한다.

```javascript
const { User } = require("./models");

// users테이블 전체를 조회해서 그 결과값을 객체로 만들어 user변수에 넣어준다.
const user = User.findAll({});

// user변수에는 조회된 결과 객체가 들어있어서, 해당 테이블 컬럼들을 조회할수 있다.
console.log(user[0].comment);
// findAll는 여러 행들을 조회하기에, 각 행들이 배열로 저장되어있다.
// 따라서 배열 인덱스로 조회한다. 첫번째 행 users테이블에 comment필드를 조회하기
```

```
SELECT * FROM users;
```

#### findone

- 쿼리 결과를 객체로 반환
  테이블의 데이터를 하나만 가져온다.

```javascript
const { User } = require("./models");

// users테이블 전체를 조회해서 그 결과값을 객체로 만들어 user변수에 넣어준다.
const user = User.findOne({});

// user변수에는 조회된 결과 객체가 들어있어서, 해당 테이블 컬럼들을 조회할수 있다.
console.log(user.comment);
// findOne는 하나만 조회된다. users테이블에 comment필드를 조회하기
```

```
SELECT * FROM users limit 1;
```

#### 조건 조회 (attributes, where)

attributes 옵션을 사용하여 원하느 컬럼만 가져올 수도 있다.
또한, where 옵션으로 조건들을 나열할 수도 있다. where 옵션은 기본적으로 AND 옵션과 같다.

```javascript
const { User } = require("./models");
const { Op } = require("sequelize");
const user = User.findAll({
  attributes: ["name", "age"],
  where: {
    married: true, // married = 1
    age: { [Op.gt]: 30 }, // age > 30;
  },
});
console.log(user.comment);
```

```
SELECT name, age FROM users WHERE married = 1 AND age > 30;
```

비교 구문이 조금 특이한데, 시퀄라이즈는 자바스크립트 객체를 사용하여 쿼리를 생성하기 때문에 특수한 연산자들이 사용된다.
![](https://velog.velcdn.com/images/sanizzang00/post/111dbc94-a81e-4795-a354-b15c70372f0d/image.png)

```javascript
const Op = Sequelize.Op
[Op.and]: [{a: 5}, {b: 6}] // (a = 5) AND (b = 6)
[Op.or]: [{a: 5}, {a: 6}] // (a = 5 OR a = 6)
[Op.gt]: 6, // > 6
[Op.gte]: 6, // >= 6
[Op.lt]: 10, // < 10
[Op.lte]: 10, // <= 10
[Op.ne]: 20, // != 20
[Op.eq]: 3, // = 3
[Op.is]: null // IS NULL
[Op.not]: true, // IS NOT TRUE
[Op.between]: [6, 10], // BETWEEN 6 AND 10
[Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
[Op.in]: [1, 2], // IN [1, 2]
[Op.notIn]: [1, 2], // NOT IN [1, 2]
[Op.like]: '%hat', // LIKE '%hat'
[Op.notLike]: '%hat' // NOT LIKE '%hat'
[Op.startsWith]: 'hat' // LIKE 'hat%'
[Op.endsWith]: 'hat' // LIKE '%hat'
[Op.substring]: 'hat' // LIKE '%hat%'
[Op.regexp]: '^[h|a|t]' // REGEXP/~ '^[h|a|t]' (MySQL/PG only)
[Op.notRegexp]: '^[h|a|t]' // NOT REGEXP/!~ '^[h|a|t]' (MySQL/PG only)
[Op.like]: { // LIKE ANY ARRAY['cat', 'hat'] - also works for iLike and notLike
[Op.any]: ['cat', 'hat']
}
[Op.gt]: { // > ALL (SELECT 1)
[Op.all]: literal('SELECT 1')
}
```

Op.or은 다음과 같이 배열 내에 적용할 쿼리들을 나열하여 사용한다.

```javascript
const user = User.findAll({
  attributes: ["name", "age"],
  where: {
    [Op.or]: [
      // married = 0 or age > 30
      { married: false },
      { age: { [Op.gt]: 30 } },
    ],
  },
});

console.log(user.comment);
```

```
SELECT name, age FROM users WHERE married = 1 OR age > 30;
```

### 데이터 넣기 (Create)

모델을 불러와 create 메서드를 사용하여 로우를 생성한다.
여기서 햇깔리는게 SQL에서의 create는 테이블을 만드는 거지만 ORM에서의 create는 insert로 쓰인다는 점을 숙지하자.

```javascript
const result = User.create({
  // 생성된 쿼리 결과를 얻는다.
  name: "beom seok",
  age: 23,
  married: false,
  comment: "안녕하세요.",
});
```

```
INSERT INTO users (name, age, married, comment)
VALUES ('beom seok', 23, 0, '안녕하세요.');
```

주의해야할 점은 데이터를 넣을 때 MySQL의 자료형이 아닌 시퀄라이즈 모델에 정의한 자료형대로 넣어야한다.
가령 married의 경우 실제 MySQL에는 TINYINT로 되어 이전엔 0을 넣어주었었지만, 현재는 BOOLEAN으로 정의돼있으므로 false를 넣어주어야 한다.

자료형 또는 옵션에 부합하지 않는 데이터를 넣을 경우 시퀄라이즈가 에러를 발생시키며, 시퀄라이즈가 알아서 MySQL 자료형으로 바꿔주니 걱정말자.

#### findOrCreate()

findOrCreate() 라는 메소드를 사용하면 조회 후에 없는 값이면 생성하고, 있는 값이면 그 값을 가져오는 형태의 오퍼레이션도 가능하다.

```javascript
// find와 create 두 조건을 이행하기 때문에 반환값이 2개 즉 배열이다.
const [user, created] = await User.findOrCreate({
  where: { username: "sdepold" },
  defaults: {
    job: "Technical Lead JavaScript",
  },
});
if (created) {
  // 만약 find하지 못하여 새로 create 될경우
  console.log(user.job); // 'Technical Lead JavaScript'
} else {
  // 만약 find가 될 경우
}
```

> where에 함께 전달되는 defaults 옵션은 검색 결과가 존재하지 않을 경우 새로 생성되는 요소가 갖는 기본값이다.

#### 정렬 (order)

정렬은 order 옵션으로 처리한다.
2차원 배열이라는 점에 주의하자. 이는 정렬은 꼭 컬럼 하나가 아닌 두 개 이상으로도 할 수 있기 때문이다.

```javascript
User.findAll({
  attributes: ["name", "age"],
  order: [
    ["age", "DESC"],
    ["name", "ASC"],
  ],
});
```

```
SELECT id, name FROM users ORDER BY age DESC name ASC;
```

#### 페이징 (limit, offset)

조회할 로우 개수는 limit으로,
조회를 시작할 로우 위치는 offset으로 할 수 있다.

```javascript
User.findAll({
  attributes: ["name", "age"],
  order: [["age", "DESC"]],
  limit: 10,
  offset: 5,
});
```

```
SELECT id, name FROM users ORDER BY age DESC LIMIT 5, 10;

SELECT id, name FROM users ORDER BY age DESC LIMIT 10 OFFSET 5;
```

#### 수정 (Update)

update 메서드로 수정할 수도 있다.
첫 번째 인수는 수정할 내용이고, 두 번째 인수는 어떤 로우를 수정할지에 대한 조건이다.
where 옵션에 조건들을 적는다.

```javascript
User.update(
  {
    comment: "새로운 코멘트.",
  },
  {
    where: { id: 2 },
  }
);
```

```
UPDATE users SET comment = '새로운 코멘트.' WHERE id = 2;
```

#### upsert()

update와 / insert 합성 버젼.
문법은 위에서 배운 findorcreate와 별 다르지 않다.

```javascript
const [city, created] = await City.upsert({
  cityName: "York",
  population: 20000,
});
console.log(created); // true or false
console.log(city); // City object
```

```
INSERT INTO Cities (cityName, population)
VALUES (?, ?)
ON DUPLICATE KEY
UPDATE
`cityName` = VALUES (`cityName`),
`population` = VALUES (`population`);
```

#### 삭제 (Delete)

로우 삭제는 destroy 메서드로 삭제한다.

```javascript
User.destroy({
  where: { id: 2 },
});
User.destroy({
  where: { id: { [Op.in]: [1, 3, 5] } },
});
```

```
DELETE FROM users WHERE id = 2;

DELETE FROM users WHERE id in(1,3,5);
```

### 시퀄라이즈 관계 쿼리문

#### include

```javascript
const user = await User.findOne({
  include: [
    {
      // join한다.
      model: Comment, // join할 테이블을 고른다.
    },
  ],
});

console.log(user.Comments[0]);
// 쿼리 결과인 user객체안에 Comments라는 키에 include한 comments테이블 쿼리들이 배열 값으로서 담겨 잇다.
// Comments 키의 이름은 User모델은 hasMany니까 복수형으로 자동으로 변환되서 생성된 것이다. (구분하기 편하게)
// => hasOne이면 단수형. M:N이면 항상 복수형.
```

```
select u.*, c.*
from users u
inner join comments c -- sequelize include는 기본동작은 inner join이다.
on u.id = c.commenter;
```

#### get 모델명

위의 include로도 충분히 관계를 맺어 쿼리를 날릴수있지만 좀더 간편한 문법으로 관계쿼리를 얻을 수 있는데, 다음과 같이 댓글에 접근할 수도 있다.
문법이 너무 추상적이라 include보다 안 와닿을수 있겠지만 ORM문법을 간소화시키고 직관적인 메소드명을 사용할수 있는 장점이 있다.

```javascript
const user = await User.findOne({});
const comments = await user.getComments();
// 따로 include할 필요없이 바로 get메소드를 쓰면 된다.
// 왜냐하면 associate로 모델을 정의한 순간 시퀄라이저가 관계가 맺어진 메소드를 알아서 만들어 주기 때문이다.

console.log(comments);
```

### SQL 쿼리 그대로 사용하기

시퀄라이즈의 쿼리를 사용하기 싫거나 헷갈린다면 직접 SQL문을 통해 쿼리할 수도 있다.

```javascript
const [result, metadata] = await sequelize.query("SELECT * FROM comments");
```
