# neo4j_workshop_basics
An introduction to the world of graph databases w/ help of Neo4j


## WORKSHOP: NEO4J BASICS
In this workshop, we will be highlighting the basic queries for working with the Cypher Query Language. 

### 1. CONSTRAINTS

/// constraint unique on movie title
```
CREATE CONSTRAINT title_uniqueness ON (m:Movie) ASSERT m.title IS UNIQUE;
```
// constraint unique on person name
```
CREATE CONSTRAINT personname_uniqueness ON (p:Person) ASSERT p.name IS UNIQUE;
```
// show constraints
```
SHOW CONSTRAINT
```
// drop constraint
```
DROP CONSTRAINT constraint_name
```

### 2. LOADING THE DATA

// load movies
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/movies.csv'
AS row
CREATE (:Movie {title: row.title, released: toInteger(row.released), tagline: row.tagline});
```

// verify
```
MATCH (m:Movie) RETURN count(*);
```

// load persons
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/people.csv'
AS row
CREATE (:Person {name: row.name, born: toInteger(row.born)});
```
// verify
```
MATCH (p:Person) RETURN count(*);
```

// load relationships (acted_in)
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/actors.csv'
AS row
MATCH (p:Person {name: row.person})
MATCH (m:Movie {title: row.movie})
MERGE (p)-[actedIn:ACTED_IN]->(m)
ON CREATE SET actedIn.roles = split(row.roles,’;’);
```

// verify
```
MATCH(p:Person {name: "Tom Hanks"})-[a:ACTED_IN]-(m:Movie) RETURN p,a,m;
```

// load relationships (directed)
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/directors.csv' AS row
MATCH (p:Person {name: row.person })
MATCH (m:Movie {title: row.movie})
MERGE (p)-[:DIRECTED]->(m);
```

// verify
```
MATCH(p:Person {name: "Tom Hanks"})-[d:DIRECTED]-(m:Movie) RETURN p,d,m;
```
// check schema
```
CALL db.schema.visualization
```
// check constraints
```
CREATE (:Movie {title: ‘The Matrix’});
```

### 3. MATCH, EXPLAIN and PROFILE

// finding Tom
```
MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

// finding Tom too
```
MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

// analyzing finding Tom
```
PROFILE MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

// analyzing finding Tom too
```
PROFILE MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

// analyzing without finding Tom
```
EXPLAIN MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

// analyzing without finding Tom too
```
EXPLAIN MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

// did Tom act with Tom ?
```
MATCH (p1:Person)-[a1:ACTED_IN]-(m:Movie)-[a2:ACTED_IN]-(p2:Person)
WHERE p1.name = "Tom Hanks"
AND p2.name = "Tom Cruise"
RETURN p1.name, a1.roles, p2.name, a2.roles, m.title;
```

// let's check 1
```
MATCH (p1:Person)-[a1:ACTED_IN]-(m:Movie)
WHERE p1.name = "Tom Hanks"
RETURN p1.name, a1.roles, m.title;
```
// let's check 2
```
MATCH (m:Movie)-[a2:ACTED_IN]-(p2:Person)
WHERE p2.name = "Tom Cruise"
RETURN p2.name, a2.roles, m.title;
```

### 4. CREATE, MERGE and DELETE

// create yourself as an actor
```
CREATE (a:Actor {name: "Kadde Oucif"});
```

// merge
```
MERGE (a:Actor {name: "Kadde Oucif"});
```

// updating properties
```
MERGE (a:Actor {name: "Kadde Oucif"})
ON MATCH SET a.job = 'actor’
RETURN a
```

// create relationship
```
MATCH (a:Actor {name: "Kadde Oucif"}),(m:Movie{title:'The Matrix'})
WITH a, m
MERGE (a)-[:ACTED_IN]->(m)
```

// delete yourself as an actor
```
MATCH(a:Actor {name: "Kadde Oucif"}) DELETE a
```

// detach delete
```
MATCH(a:Actor {name: "Kadde Oucif"}) DETACH DELETE a
```

// delete relationship
```
MATCH (a:Actor {name:"Kadde Oucif"})-[r]->(m:Movie {title: "The Matrix"})
DELETE r
```
### 5. TRAVERSAL

// single MATCH
```
MATCH (valKilmer:Person)-[:ACTED_IN]->(m:Movie),
	    (actor:Person)-[:ACTED_IN]->(m)
WHERE valKilmer.name = 'Val Kilmer’
RETURN m.title as movie , actor.name
```

// multiple MATCH
```
MATCH (valKilmer:Person)-[:ACTED_IN]->(m:Movie) 
MATCH (actor:Person)-[:ACTED_IN]->(m)
WHERE valKilmer.name = 'Val Kilmer’ 
RETURN m.title as movie , actor.name
```

// multiple anchors
```
MATCH (p1:Person)-[:ACTED_IN]->(m) MATCH (n)<-[:ACTED_IN]-(p2:Person) 
WHERE p1.name = ’Tom Hanks’ 
  AND p2.name = ’Tom Cruise’  
  AND m=n 
RETURN m.title
```

// returning a subgraph
```
MATCH paths = (m:Movie)-[rel]-(p:Person)
WHERE m.title = 'The Replacements'
RETURN paths
```
### 6. COLLECT and COUNT

// aggregation using collect()
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name ='Tom Cruise'
RETURN p.name, collect(m.title) AS `movies`
```

// aggregation using count()
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
RETURN a.name, d.name, count(m)
```

// check
```
MATCH path=(a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
WHERE a.name='Ben Miles' AND d.name='James Marshall'
RETURN path
```
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
RETURN m.title, collect(a.name)[1] AS `A cast member`, size(collect(a.name)) AS castSize
```

### 7. WITH and UNWIND

// with and unwind
```
MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
WITH collect(p) AS actors, count(p) AS actorCount, m
UNWIND actors AS actor
RETURN m.title, actorCount, actor.name
```

### 8. DISTINCT, ORDER BY and LIMIT

// eliminate duplicates using DISTINCT
```
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN DISTINCT m.title, m.released
```

// eliminate duplicates using DISTINCT in lists
```
MATCH (p:Person)-[:ACTED_IN | DIRECTED | WROTE]->(m:Movie)
WHERE m.released = 1996
RETURN m.title, collect(DISTINCT p.name) AS credits
```

// ordering results
```
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks' OR p.name = 'Keanu Reeves'
RETURN DISTINCT m.title, m.released 
ORDER BY m.released DESC, m.title
```

// limiting the number of results
```
MATCH (m:Movie)
RETURN m.title as title, m.released as year   
ORDER BY m.released DESC LIMIT 10
```

### 9. ADDITIONAL CYPHERS QUERIES

// find movies released in the '90s
```
MATCH (nineties:Movie) 
WHERE nineties.released >= 1990 AND nineties.released < 2000
RETURN nineties.title
```

// the bacon path, the shortest path of any relationships to Meg Ryan
```
MATCH p=shortestPath(
(bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"}))
RETURN p
```

// retrieve all movies that wew released in the years 2000, 2004, and 2008, returning title and year
```
MATCH (m:Movie)
WHERE m.released IN [2000, 2004, 2008]
RETURN m.title, m.released
```

// retrieve all persons who’s name begins with Tom and optionally return the name of a movie that this person directed.
```
MATCH (p:Person)
WHERE p.name STARTS WITH 'Tom'
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
```

// retrieve all actors that have not appeared in more than 3 movies. Return their names and list of movies.
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(a) AS numMovies, collect(m.title) AS movies
WHERE numMovies <= 5
RETURN a.name, movies
```

### [BONUS] EASTER EGG
```
MATCH (p:Person{name:'Emil Eifrem'})-[a:ACTED_IN]->(m:Movie)
RETURN p, a, m
```

### Additional links

// course - introduction to Neo4j
```
:play 4.0-intro-neo4j-exercises
```
