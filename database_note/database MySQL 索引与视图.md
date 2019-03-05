# database MySQL 索引与视图
## 索引



## 视图
视图：</br>
使用一个别名来表示一系列复杂的SQL查询语句。在一下次查询时，则不再需要重复输入SQL语句而直接使用这个别名来进行查询即可。</br>
* 视图仅用于简化查询语句，视图不包含表中的真实数据；</br>
* 视图可以嵌套，可使用根据一个视图的查询结果来设计另外一个视图，但是有可能造成性能下降；</br>
* 每个视图必须有唯一的视图名，不能与其他视图以及表的名字重复；</br>
* 视图不但简化了SQL语句，让SQL语句获得重复使用，还可以保护数据，可给不同角色设置不同的数据查询权限；</br>
* 视图可返回与底层表表示格式不同的数据，但视图不能使用索引，也不能有关联的触发器和默认值；</br>
* 每种数据库对视图使用所定义的规则都有所不同，使用前应查询数据库的详细文档。</br>

### 创建与删除视图
创建视图
```
CREATE VIEW viewname AS
具体详细的SQL语句;
```
删除视图
```
DROP VIEW viewname;
```
注意：貌似是没有直接修改已有视图的方法的，仅能删除旧视图后再新建一个新的视图。</br>

### 根据视图查询数据
```
SELECT 要查询的数据
FROM 视图名
WHERE 过滤条件
```

### 视图实例
比如 actor、film_actor、film 三个表，查询某部电影的演员名单。</br>
首先，查询每部电影的演员名单：
```
SELECT film.title, actor.first_name, actor.last_name 
FROM sakila.actor, sakila.film, sakila.film_actor
where film.film_id = film_actor.film_id AND film_actor.actor_id = actor.actor_id;
```
给这个语句创建一个视图，注意创建视图是没有输出的：</br>
```
CREATE VIEW FilmActorList AS
SELECT film.title, actor.first_name, actor.last_name 
FROM sakila.actor, sakila.film, sakila.film_actor
where film.film_id = film_actor.film_id AND film_actor.actor_id = actor.actor_id;
```
使用这个视图来对特定某部电影进行查询：
```
SELECT title, first_name, last_name
FROM FilmActorList
WHERE title = 'ANGELS LIFE';
```
不使用视图时的查询语句：
```
SELECT film.title, actor.first_name, actor.last_name 
FROM sakila.actor, sakila.film, sakila.film_actor
where film.title = 'ANGELS LIFE' AND film.film_id = film_actor.film_id AND film_actor.actor_id = actor.actor_id;
```
注意：</br>
视图查询时column的写法，视图相当于一个已按视图指代的SQL的WHERE条件整合过的查询结果，所以使用column再次查询的时候是不需要再特别指出column真实所在的表名的。即是要根据视图所指代的SQL语句的输出来确定视图查询的SQL写法，也即是说可以把视图作为一个新表来看待，可是这个表只用于查询而不包含数据本身。</br>
