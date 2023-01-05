#### 函数
```sql
CREATE OR REPLACE FUNCTION "organization"."get_authority"("id" uuid)
  RETURNS SETOF "pg_catalog"."text" AS $BODY$
SELECT api_code
FROM admin.rel_permission_api
WHERE permission_code IN (SELECT permission_code
                          FROM organization.rel_role_permission
                          WHERE role_id IN (SELECT role_id
                                            FROM organization.rel_user_role
                                            WHERE user_id = id));
$BODY$
  LANGUAGE sql VOLATILE
  COST 100
  ROWS 1000;

ALTER FUNCTION "organization"."get_authority"("id" uuid) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."get_authorized_menu"("user_id" uuid)
  RETURNS SETOF "admin"."menu" AS $BODY$
WITH RECURSIVE x AS (SELECT id, parent_id
                     FROM admin.menu
                     WHERE id IN (
                         -- 权限直接对应的菜单
                         SELECT menu_id as id
                         FROM admin.rel_permission_menu
                         WHERE permission_code IN (
                             -- 用户的权限
                             SELECT *
                             FROM organization.get_authorized_permission(user_id)))
                     UNION ALL
                     SELECT m.id, m.parent_id
                     FROM admin.menu m
                              JOIN x ON m.id = x.parent_id)
SELECT *
FROM admin.menu
WHERE
   -- 普通用户
    id IN (SELECT id FROM x)
   OR
   -- 管理员
        EXISTS(
                SELECT *
                FROM organization.rel_user_role
                WHERE role_id = '00000000-0000-0000-0000-000000000000'
                  AND rel_user_role.user_id = get_authorized_menu.user_id)
        and admin.menu.is_enable = true
ORDER BY id ASC;
$BODY$
  LANGUAGE sql VOLATILE
  COST 100
  ROWS 1000;

ALTER FUNCTION "organization"."get_authorized_menu"("user_id" uuid) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."get_authorized_operation"("user_id" uuid)
  RETURNS SETOF "admin"."operation" AS $BODY$
SELECT *
FROM admin.operation
WHERE code IN (SELECT operation_code
               FROM admin.rel_permission_operation
               WHERE permission_code IN (SELECT *
                                         FROM organization.get_authorized_permission(user_id)));
$BODY$
  LANGUAGE sql VOLATILE
  COST 100
  ROWS 1000;

ALTER FUNCTION "organization"."get_authorized_operation"("user_id" uuid) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."get_authorized_permission"("id" uuid)
  RETURNS SETOF "pg_catalog"."text" AS $BODY$
SELECT permission_code
FROM organization.rel_role_permission
WHERE role_id IN (SELECT role_id
                  FROM organization.rel_user_role
                  WHERE user_id = id);
$BODY$
  LANGUAGE sql VOLATILE
  COST 100
  ROWS 1000;

ALTER FUNCTION "organization"."get_authorized_permission"("id" uuid) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."set_role_permission"("role" uuid, "permissions" _text)
  RETURNS "pg_catalog"."void" AS $BODY$
DELETE
FROM organization.rel_role_permission
WHERE role_id = role;
INSERT INTO organization.rel_role_permission
SELECT role, unnest(permissions);
$BODY$
  LANGUAGE sql VOLATILE
  COST 100;

ALTER FUNCTION "organization"."set_role_permission"("role" uuid, "permissions" _text) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."set_user_role"("uid" uuid, "roles" _text)
  RETURNS "pg_catalog"."void" AS $BODY$
DELETE
FROM organization.rel_user_role
WHERE user_id = uid;
INSERT INTO organization.rel_user_role
SELECT uid, unnest(roles) ::uuid;
$BODY$
  LANGUAGE sql VOLATILE
  COST 100;

ALTER FUNCTION "organization"."set_user_role"("uid" uuid, "roles" _text) OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."sync_dingtalk_departments"()
  RETURNS "pg_catalog"."void" AS $BODY$
    -- 将钉钉中的部门同步到内部通讯录
INSERT INTO organization.department (name, sort, dingtalk_id)
SELECT name, "order", dept_id
FROM cookroom.dingtalk_department ON CONFLICT (dingtalk_id) DO
UPDATE
    SET name = excluded.name,
    sort = excluded.sort,
    dingtalk_id = excluded.dingtalk_id;

-- 刷新各部门的上级部门信息
WITH m AS (SELECT od.id, op.id parent_id
           FROM organization.department od
                    JOIN cookroom.dingtalk_department cd ON od.dingtalk_id = cd.dept_id
                    JOIN cookroom.dingtalk_department cp ON cd.parent_id = cp.dept_id
                    JOIN organization.department op ON cp.dept_id = op.dingtalk_id)
UPDATE organization.department d
SET parent_id = m.parent_id FROM m
WHERE d.id = m.id;

-- 将过期的部门标记为失效
UPDATE organization.department
SET tz_retire = NOW()
WHERE dingtalk_id NOT IN (SELECT dept_id FROM cookroom.dingtalk_department);
$BODY$
  LANGUAGE sql VOLATILE
  COST 100;

ALTER FUNCTION "organization"."sync_dingtalk_departments"() OWNER TO "postgres";

CREATE OR REPLACE FUNCTION "organization"."sync_dingtalk_users"()
  RETURNS "pg_catalog"."void" AS $BODY$
-- 同步钉钉用户到内部通讯录
INSERT INTO organization.user (name, mobile, avatar, dingtalk_id, job_number, sort)
SELECT name, mobile, avatar, userid, CASE WHEN job_number = '' THEN null ELSE job_number END, dept_order
FROM cookroom.dingtalk_user
WHERE userid NOT IN (SELECT job_number FROM cookroom.dingtalk_user WHERE job_number is NOT NULL GROUP BY job_number HAVING count(*) > 1) ON CONFLICT (dingtalk_id)
    DO
UPDATE
    SET name = excluded.name,
    mobile = excluded.mobile,
    avatar = excluded.avatar,
    job_number = excluded.job_number,
    sort = excluded.sort;

-- 清除原有的部门关系
DELETE
FROM organization.rel_user_department;

-- 更新人员所在部门信息
WITH ex AS (
    -- 将人员表里的单位数组展开成1对1关系
    SELECT userid, unnest(dept_id_list) dept_id FROM cookroom.dingtalk_user)
INSERT
INTO organization.rel_user_department(user_id, department_id, is_primary)
SELECT ou.id, od.id, true -- TODO 将所有部门都作为主要科室，管理员可以通过后台指定
FROM ex -- 通过外联将钉钉里的编号转换成系统内部编号
         JOIN organization.department od ON ex.dept_id = od.dingtalk_id
         JOIN organization.user ou ON ex.userid = ou.dingtalk_id;

-- 将过期的人员标记为失效
UPDATE organization.user
SET tz_retire = NOW()
WHERE tz_retire IS NOT NULL
  AND dingtalk_id NOT IN (SELECT userid FROM cookroom.dingtalk_user);
$BODY$
  LANGUAGE sql VOLATILE
  COST 100;

ALTER FUNCTION "organization"."sync_dingtalk_users"() OWNER TO "postgres";
```
