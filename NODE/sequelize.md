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
