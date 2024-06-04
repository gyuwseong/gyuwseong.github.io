---
title: "TypeORM 적응기 (feat.custom repository)"
description:
date: 2024-06-04
update: 2024-06-04
tags:
  - 조각글
---

개인 코드 스타일을 드러낼 작은 프로젝트를 진행하면서, 기존에 사용했던 [Sequelize](https://sequelize.org) 대신 [Typeorm](https://typeorm.io) 을 적용해 보고 있다.

아직 시작한 지 얼마 되지 않아서 적응 중이고, 딥한 레벨의 로직을 구현하지 않았기 때문에 아주 큰 차이점을 느끼진 못했지만, ORM 에서 레이어 분리를 위해 자주 사용하는 repository 를 구현할 때의 차이점을 기록해 보는 포스팅이다.

## **Sequelize custom repository**

```typescript
// UserRepository.ts

@Injectable()
export class UserRepository {
  constructor(@InjectModel(User) private readonly repository: typeof User) {}

  async findUserById(id: number): Promise<User> {
    return this.repository.findByPk(id);
  }
}
```

- sequelize 에서는 `@Injectable()` 데코레이터를 통해 `UserRepository` 클래스를 NestJS 컨테이너에 등록하고 관리한다.
- 클래스의 생성자에서 `@InjectModel(User)` 데코레이터를 사용하여 User 모델을 주입받는다.

&nbsp;

```typescript
// UserModel.ts

@Table({ tableName: 'user' })
export class User extends Model
```

- 이때 `@Table` 데코레이터를 통해 `User` 테이블과 `User` 모델이 맵핑되기 때문에 User 모델을 주입받을 수 있다.

&nbsp;

```typescript
// UserModule.ts

@Module({
  imports: [SequelizeModule.forFeature([User])],
  providers: [UserRepository],
})
export class UseRepositoryModule {}
```

- `SequelizeModule.forFeature` 를 통해 repository 를 NestJS 모듈 DI 에 등록하면 다른 서비스나 모듈에서도 해당 repository 를 주입받아 사용할 수 있다.

&nbsp;

&nbsp;

## **TypeORM custom repository**

```typescript
// UserRepository.ts

@Injectable()
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  async findOneById(id: number): Promise<User> {
    return this.findOneBy({ id });
  }
}
```

해당 방식은 [여기](https://gist.github.com/anchan828/9e569f076e7bc18daf21c652f7c3d012?permalink_comment_id=4319458#gistcomment-4319458)에서 찾은 방식인데, 본문에 소개된 `@CustomRepository` 방식보다 더 직관적이고 깔끔해서 이 방식으로 적용했다.

- `Repository` 클래스를 상속받는 자식 클래스에서 `DataSource` 를 주입 받고, 생성자를 호출하면서 `UserRepository` 를 생성한다.

- TypeORM 은 repository 클래스를 생성 시 `EntityTarget` 과 `EntityManager` 를 생성하는데, `EntityTarget` 은 클래스가 다룰 타겟 `entity` 이고, `EntityManager` 는 데이터베이스 통신 작업을 수행하는 컴포넌트이다.

- `DataSource` 는 데이터베이스 연결 및 `EntityManager` 생성을 관리하기 때문에 `DataSource` 를 주입받아서 `Custom Repository` 와 `EntityManager` 를 생성하고, 해당 `EntityManager` 를 통해 데이터베이스 작업을 수행하는 방식으로 동작한다.

&nbsp;

```typescript
// UserModule.ts

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UserRepository],
})
export class UserModule {}
```

- `TypeOrmModule.forFeature` 를 통해 repository 를 NestJS 모듈 DI 에 등록하면 다른 서비스나 모듈에서도 해당 repository 를 주입받아 사용할 수 있다.
