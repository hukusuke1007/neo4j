# Cheet sheet


# Command

## Create
```sh
# idが重複しないよう制約
CREATE CONSTRAINT ON (u:User) ASSERT u.id IS UNIQUE

# 制約解除
DROP CONSTRAINT ON (u:User) ASSERT u.id IS UNIQUE
```

```sh
# n: ノード, User: ラベル, User{...}: プロパティ 
CREATE (n:User{user_id:100, user_name:'Alice'})


# ノードに対して複数のタグをつけることができる
# User, Woman, Engineerがタグ
CREATE (:User:Woman:Engineer{user_id:100, user_name:'Alice'})

# (u1)-[:follows]->(u2): リレーション u1がu2をフォローしている関係
CREATE (u1:User:Woman:Engineer{user_id:100, user_name:'Alice'})
CREATE (u2:User:Man:SIer{user_id:101, user_name:'Bob'})
CREATE (u1)-[:follows]->(u2)

# Alice -> Bob -> Carlのフォロー順に作成
# CREATEは , で省略できる
CREATE (u1:User{user_id:100, user_name:'Alice'}),
    (u2:User{user_id:102, user_name:'Bob'}),
    (u3:User{user_id:103, user_name:'Carl'}),
    (u1)-[:follows]->(u2)-[:follows]->(u3)
```

## Read

```sh
# 全てのノードを取得
MATCH (n) return n

# 送信元のノード取得
MATCH (n)-[]->() 
RETURN n

MATCH (u1:User{user_name:'Alice'})-[f:follows]->(u2:User{user_name:'Bob'})
RETURN u1, u2, f

# 間にパスが挟まっているパスの検索
MATCH (u1:User)-[*2]-(u2:User)
RETURN u1, u2

# WHERE
MATCH (n)
WHERE n.user_id = 102
RETURN n
```

## Delete

```sh
# 全てのノードを削除
MATCH (n) DETACH DELETE n

# フォロー関係のノードとリレーションを削除
MATCH (u1:User)-[f:follows]-(u2:User)
DELETE f, u1, u2
```

## 最短経路問題

```sh
# 作成
CREATE (p1:Point{name:"p1"}),
     (p2:Point{name:"p2"}),
     (p3:Point{name:"p3"}),
     (p4:Point{name:"p4"}),
     (p5:Point{name:"p5"}),
     (p1)-[:Route{weight:7}]->(p2),
     (p1)-[:Route{weight:5}]->(p3),
     (p1)-[:Route{weight:8}]->(p4),
     (p1)-[:Route{weight:9}]->(p5),
     (p2)-[:Route{weight:3}]->(p3),
     (p2)-[:Route{weight:2}]->(p4),
     (p2)-[:Route{weight:1}]->(p5),
     (p3)-[:Route{weight:4}]->(p4),
     (p3)-[:Route{weight:7}]->(p5),
     (p4)-[:Route{weight:9}]->(p5)

# p1からp5の最短経路（weightは考えない）
MATCH (p1:Point{name:"p1"}),
     (p5:Point{name:"p5"}),
     g = shortestPath((p1)-[:Route*]-(p5))
RETURN g

# p1からp5の最短経路（weightが最短となる）
MATCH g = (:Point{name:"p1"})-[:Route*]-(:Point{name:"p5"})
RETURN g,
     REDUCE(weight=0, r in RELATIONSHIPS(g) | weight+r.weight) AS score
ORDER BY score ASC
LIMIT 1

# p1からp5の最短経路（全て経由する且つweightが最短となる）
MATCH g = (:Point{name:"p1"})-[:Route*4]-(:Point{name:"p5"})
WHERE ALL(n IN NODES(g) WHERE SINGLE(m IN NODES(g) WHERE m = n))
RETURN g,
     REDUCE(weight=0, r IN RELATIONSHIPS(g) | weight+r.weight) AS score
ORDER BY score ASC
LIMIT 1
```


## Reference

[https://neo4j.com/docs/cypher-manual/current/introduction/](https://neo4j.com/docs/cypher-manual/current/introduction/)