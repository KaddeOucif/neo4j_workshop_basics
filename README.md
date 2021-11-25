![neo4j-banner](pics/neo4j_banner.png)
## WORKSHOP: NEO4J BASICS
An introduction to neo4js graph database. In this workshop, we will be highlighting the basic queries for working with the Cypher Query Language. 

## Prerequisites
Before starting, make sure you got [Neo4j Desktop](https://neo4j.com/download/) downloaded. You can also use the [Neo4j Sandbox](https://neo4j.com/sandbox/), but please use it as a last resort as you'd only have access for a couple of days. 

<details>
	<summary> <b> 1. CONSTRAINTS </b></summary>
	<br>

Let's start with creating constraints within our database. The first constraint is to make sure nobody creates multiple movies with the same title.
```
CREATE CONSTRAINT title_uniqueness ON (m:Movie) ASSERT m.title IS UNIQUE;
```
Let's do the same thing with person names. In the real world, multiple persons could have the same name, but we do not have any personal identifiers in the database as of right now - so let's just pretend that this is how it works.
```
CREATE CONSTRAINT personname_uniqueness ON (p:Person) ASSERT p.name IS UNIQUE;
```
Now let's have a look at our constraints set on this database.
```
SHOW CONSTRAINT
```
To drop a constraint we write:
```
DROP CONSTRAINT <constraint_name>
```
</details>

<details>
	<summary> <b> 2. LOADING THE DATA </b></summary>
	<br>

Now let's load movies into the database from an external data source. This data source provides us with the movie nodes from a .csv that we will be ingesting into our database.
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/movies.csv'
AS row
CREATE (:Movie {title: row.title, released: toInteger(row.released), tagline: row.tagline});
```

Let's verify that our movies got imported, you should have around 38 movies after executing this query
```
MATCH (m:Movie) RETURN count(*);
```

Now let's do the same thing but for persons
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/people.csv'
AS row
CREATE (:Person {name: row.name, born: toInteger(row.born)});
```
Let's verify that the persons got into the database as well
```
MATCH (p:Person) RETURN count(*);
```

If everything went well, then we should move on to the next step which would be to create a relationship between a Person and a Movie. In this case we're going to make sure we create relationships between the Actors and the Movies (ACTED_IN).
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/actors.csv'
AS row
MATCH (p:Person {name: row.person})
MATCH (m:Movie {title: row.movie})
MERGE (p)-[actedIn:ACTED_IN]->(m)
ON CREATE SET actedIn.roles = split(row.roles,';');
```

Let's verify that we managed to create the relationships.
```
MATCH(p:Person {name: "Tom Hanks"})-[a:ACTED_IN]-(m:Movie) RETURN p,a,m;
```

Let's create another relationship, but instead of actors we make sure that those who directed certain movies have a DIRECTED relationship set up.
```
LOAD CSV WITH HEADERS FROM 'http://data.neo4j.com/intro/movies/directors.csv' AS row
MATCH (p:Person {name: row.person })
MATCH (m:Movie {title: row.movie})
MERGE (p)-[:DIRECTED]->(m);
```

Let's see if our query managed to create the DIRECTED relationship.
```
MATCH(p:Person {name: "Tom Hanks"})-[d:DIRECTED]-(m:Movie) RETURN p,d,m;
```
By doing all this, we actually have a schema to look at. We can see what the schema looks like by calling; 
```
CALL db.schema.visualization
```
Lastly, let's see if our constraints are working as they should. If you remember in the beginning, we made sure that no duplicate movie could be created in the graph database. If you run the query below, then you should get an error.
```
CREATE (:Movie {title: 'The Matrix'});
```
</details>
	
<details>
	<summary> <b> 3. MATCH, EXPLAIN and PROFILE </b></summary>
	<br>

By finding a certain node we run the MATCH command, which can be followed by a certain criteria (such as name in this case).
```
MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

Another way to do this could be:
```
MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

If you'd like to analyze your queries then you can run:
```
PROFILE MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

Let's analyze the other query as well:
```
PROFILE MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

<b>PROFILE</b> analyzes the query while executing it. If you'd like to just analyze the query without executing it then <b>EXPLAIN</b> is your command.
```
EXPLAIN MATCH (p:Person {name: "Tom Hanks"})
RETURN p;
```

Let's do the same thing here as well:
```
EXPLAIN MATCH (p:Person)
WHERE p.name = "Tom Hanks" RETURN p;
```

Let's further elaborate on the MATCH clauses. Did Tom act with Tom?
```
MATCH (p1:Person)-[a1:ACTED_IN]-(m:Movie)-[a2:ACTED_IN]-(p2:Person)
WHERE p1.name = "Tom Hanks"
AND p2.name = "Tom Cruise"
RETURN p1.name, a1.roles, p2.name, a2.roles, m.title;
```

Interesting. Let's confirm the previous result by checking what roles Tom Hanks have had.
```
MATCH (p1:Person)-[a1:ACTED_IN]-(m:Movie)
WHERE p1.name = "Tom Hanks"
RETURN p1.name, a1.roles, m.title;
```

And now Tom Cruise.
```
MATCH (m:Movie)-[a2:ACTED_IN]-(p2:Person)
WHERE p2.name = "Tom Cruise"
RETURN p2.name, a2.roles, m.title;
```
</details>

<details>
	<summary> <b> 4. CREATE, MERGE and DELETE </b></summary>
	<br>

To create a node we use the CREATE command. Replace the name below with your name.
```
CREATE (a:Actor {name: "Kadde Oucif"});
```

We can do the same thing with MERGE, however MERGE always checks if the specific object we're trying to create exists. In this way, it's helpful to think of MERGE as attempting a MATCH on the pattern, and if no match is found, a CREATE of the pattern.
```
MERGE (a:Actor {name: "Kadde Oucif"});
```

Let's use MERGE to create a job description in our Actor node.
```
MERGE (a:Actor {name: "Kadde Oucif"})
ON MATCH SET a.job = 'actor'
RETURN a
```

Now let's use our actor node and create a relationship to a movie.
```
MATCH (a:Actor {name: "Kadde Oucif"}),(m:Movie{title:'The Matrix'})
WITH a, m
MERGE (a)-[:ACTED_IN]->(m)
```

Nice. Now let's try to delete the node we created previously
```
MATCH(a:Actor {name: "Kadde Oucif"}) DELETE a
```

As you can see, this wont work since your node is connected to other nodes. let's try a detach delete then.
```
MATCH(a:Actor {name: "Kadde Oucif"}) DETACH DELETE a
```

That worked better. But what if I only want to delete a relationship? Let's recreate the Actor-node and link it to a movie
```
CREATE (a:Actor {name: "Kadde Oucif"})
WITH a
MATCH (m:Movie {title: "The Matrix"})
MERGE (a)-[:ACTED_IN]->(m)
```

Now let's see how we can delete a relationship
```
MATCH (a:Actor {name:"Kadde Oucif"})-[r]->(m:Movie {title: "The Matrix"})
DELETE r
```

Let's check if the relationship is still there
```
MATCH (a:Actor {name: "Kadde Oucif"})
MATCH (m:Movie {title: "The Matrix"})
RETURN a, m
```
</details>

<details>
	<summary> <b> 5. TRAVERSAL </b></summary>
	<br>

There's different ways we can traverse the data. Here's a single MATCH.
```
MATCH (valKilmer:Person)-[:ACTED_IN]->(m:Movie),
	    (actor:Person)-[:ACTED_IN]->(m)
WHERE valKilmer.name = 'Val Kilmer'
RETURN m.title as movie , actor.name as actor
```

Here's multiple:
```
MATCH (valKilmer:Person)-[:ACTED_IN]->(m:Movie) 
MATCH (actor:Person)-[:ACTED_IN]->(m)
WHERE valKilmer.name = 'Val Kilmer'
RETURN m.title as movie , actor.name
```

Here we have multiple anchors.
```
MATCH (p1:Person)-[:ACTED_IN]->(m) MATCH (n)<-[:ACTED_IN]-(p2:Person) 
WHERE p1.name = 'Tom Hanks'
  AND p2.name = 'Meg Ryan'
  AND m=n 
RETURN m.title
```

Now let's return a subgraph.
```
MATCH paths = (m:Movie)-[rel]-(p:Person)
WHERE m.title = 'The Replacements'
RETURN paths
```
								      
</details>	

<details>
	<summary> <b> 6. COLLECT and COUNT </b></summary>
	<br>

The function collect() returns a single aggregated list containing the values returned by an expression.
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name ='Tom Cruise'
RETURN p.name, collect(m.title) AS `movies`
```

The function count() returns the number of values or rows.
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
RETURN a.name, d.name, count(m) as amountOfMoviesTogether
```

Let's check the results here, apparently the previous query returned 2 movies.
```
MATCH path=(a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
WHERE a.name='Ben Miles' AND d.name='James Marshall'
RETURN path
```

Here's a way to aggregation to get cast size & first cast members
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
RETURN m.title, collect(a.name)[1] AS `A cast member`, size(collect(a.name)) AS castSize
```
</details>
	
<details>
	<summary> <b> 7. WITH and UNWIND </b></summary>
	<br>

The WITH clause allows query parts to be chained together, piping the results from one to be used as starting points or criteria in the next. With UNWIND, you can transform any list back into individual rows. These lists can be parameters that were passed in, previously collect -ed result or other list expressions.
```
MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
WITH collect(p) AS actors, count(p) AS actorCount, m
UNWIND actors AS actor
RETURN m.title, actorCount, actor.name
```
</details>

<details>
	<summary> <b> 8. DISTINCT, ORDER BY and LIMIT</b></summary>
	<br>

Eliminate duplicates using DISTINCT
```
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN DISTINCT m.title, m.released
```
	
Eliminate duplicates using DISTINCT in lists, f.e. Tom Hanks acted and directed "That Thing You Do"
```
MATCH (p:Person)-[:ACTED_IN | DIRECTED | WROTE]->(m:Movie)
WHERE m.released = 1996
RETURN m.title, collect(DISTINCT p.name) AS credits
```

Return your results in a certain order.
```
MATCH (p:Person)-[:DIRECTED | ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks' OR p.name = 'Keanu Reeves'
RETURN DISTINCT m.title, m.released 
ORDER BY m.released DESC, m.title
```

Limiting the number of results
```
MATCH (m:Movie)
RETURN m.title as title, m.released as year   
ORDER BY m.released DESC LIMIT 10
```
</details>

<details>
	<summary> <b> 9. ADDITIONAL CYPHERS QUERIES</b></summary>
	<br>

Find movies released in the '90s.
```
MATCH (nineties:Movie) 
WHERE nineties.released >= 1990 AND nineties.released < 2000
RETURN nineties.title
```

The bacon path, the shortest path of any relationships to Meg Ryan.
```
MATCH p=shortestPath(
(bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"}))
RETURN p
```

Retrieve all movies that was released in the years 2000, 2004, and 2008, returning title and year.
```
MATCH (m:Movie)
WHERE m.released IN [2000, 2004, 2008]
RETURN m.title, m.released
```

Retrieve all persons who’s name begins with Tom and optionally return the name of a movie that this person directed.
```
MATCH (p:Person)
WHERE p.name STARTS WITH 'Tom'
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
```

Retrieve all actors that have not appeared in more than 3 movies. Return their names and list of movies.
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(a) AS numMovies, collect(m.title) AS movies
WHERE numMovies <= 5
RETURN a.name, movies
```
</details>

<details>
	<summary> <b> [BONUS] EASTER EGG</b></summary>
	<br>
	
```
MATCH (p:Person{name:'Emil Eifrem'})-[a:ACTED_IN]->(m:Movie)
RETURN p, a, m
```
	
</details>


### Additional links

A more thorough course / introduction can be found by visiting the play guide in your Neo4j Browser.
```
:play 4.0-intro-neo4j-exercises
```
