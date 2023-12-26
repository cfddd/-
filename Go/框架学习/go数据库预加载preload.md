> [预加载 | GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/docs/preload.html)
## 预加载示例
GORM允许使用 Preload通过多个SQL中来直接加载关系, 例如：

```
type User struct {
  gorm.Model
  Username string
  Orders   []Order
}

type Order struct {
  gorm.Model
  UserID uint
  Price  float64
}

// 查找 user 时预加载相关 Order
db.Preload("Orders").Find(&users)
// SELECT * FROM users;
// SELECT * FROM orders WHERE user_id IN (1,2,3,4);

db.Preload("Orders").Preload("Profile").Preload("Role").Find(&users)
// SELECT * FROM users;
// SELECT * FROM orders WHERE user_id IN (1,2,3,4); // has many
// SELECT * FROM profiles WHERE user_id IN (1,2,3,4); // has one
// SELECT * FROM roles WHERE id IN (4,5,6); // belongs to
```
## Joins 预加载
Preload loads the association data in a separate query, Join Preload will loads association data using left join, for example:

```
db.Joins("Company").Joins("Manager").Joins("Account").First(&user, 1)
db.Joins("Company").Joins("Manager").Joins("Account").First(&user, "users.name = ?", "jinzhu")
db.Joins("Company").Joins("Manager").Joins("Account").Find(&users, "users.id IN ?", []int{1,2,3,4,5})
```
带条件的 Join

```
db.Joins("Company", DB.Where(&Company{Alive: true})).Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id` AND `Company`.`alive` = true;
```
Join nested model

```
db.Joins("Manager").Joins("Manager.Company").Find(&users)
// SELECT "users"."id","users"."created_at","users"."updated_at","users"."deleted_at","users"."name","users"."age","users"."birthday","users"."company_id","users"."manager_id","users"."active","Manager"."id" AS "Manager__id","Manager"."created_at" AS "Manager__created_at","Manager"."updated_at" AS "Manager__updated_at","Manager"."deleted_at" AS "Manager__deleted_at","Manager"."name" AS "Manager__name","Manager"."age" AS "Manager__age","Manager"."birthday" AS "Manager__birthday","Manager"."company_id" AS "Manager__company_id","Manager"."manager_id" AS "Manager__manager_id","Manager"."active" AS "Manager__active","Manager__Company"."id" AS "Manager__Company__id","Manager__Company"."name" AS "Manager__Company__name" FROM "users" LEFT JOIN "users" "Manager" ON "users"."manager_id" = "Manager"."id" AND "Manager"."deleted_at" IS NULL LEFT JOIN "companies" "Manager__Company" ON "Manager"."company_id" = "Manager__Company"."id" WHERE "users"."deleted_at" IS NULL

