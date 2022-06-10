---
layout: post
title: Rhea's Galaxy Map
description: Adventures in DAGs, Graph Databases, and Brownian Motion
---

In the [last post](/introducing-rhea), I covered the various game titles and mechanics they utilize that I've drawn inspiration from for the design of Rhea. In covering Rhea's galactic map, I'd like to focus upon two of them heavily-- Eve Online and Stellaris.

### Inspiration

Eve Online and Stellaris have very similar map systems. Both have solar systems represented in a DAG-like structure. The DAG's nodes represent solar systems. In the case of Eve Online, this structure's edges are representative of Stargates, and in Stellaris, warp lanes between solar systems.

In any case, both systems function in a similar way, in that players and their fleets traverse a large DAG to move around the galaxy.

#### Stellaris

Based on a system of ephemeral matches, Stellaris's game maps are generated uniquely for each. Thus, each time the player creates a new empire or species and starts a game, a new map is generated and configured via the player's selected settings when creating the game.

One of the major strategies when starting a game of Stellaris is to rapidly expand to claim and build defenses around choke-point systems, or systems that have many connections as to control the flow of traffic through that portion of the map. This is a side-effect of this galactic map model that I'm intent upon creating.

![](https://s3.amazonaws.com/images.willricketts.com/rhea/stellarismap.jpg)

#### Eve Online

Eve's map is persistent, in that the game has no concept of "rounds" or "matches." The game map, save for additions by their publisher, has and will remain the same since the release of the game in 2003. Being that the intent of Rhea is to be a single persistent game world, this is the path I've elected to follow.

Some very desirable but non-obvious effects arise from this kind of model for a galactic map. With a static map, various regions or sections of it become known within the playerbase for being better or worse to own from a strategic perspective. Eve has regions of the game that are notoriously difficult to invade, and has a few that are notoriously favorable for an invading military to stage their assets.

![](https://s3.amazonaws.com/images.willricketts.com/rhea/evemap.png)

### Generating the Map

Creating a map of this size and detail was a challenging problem to solve, and at the time of this writing, still presents obstacles to overcome to achieve an optimal map structure.

#### First Attempts

When I first sat down to work on figuring out the most immediate challenges in building Rhea, my intent was to get something simple working and then expand upon the idea later. Upon researching how I would go about generating a map of this nature with DAG-like properties, I found [Astrosynthesis](https://www.nbos.com/products/astrosynthesis), a piece of software used for generating celestial structures primarily for role-playing games and hobbyist world-building. This program had a bunch of great features that'd be useful for my needs, but two features stood out in particular. Astrosynthesis has a user-created plugin ecosystem and an XML exporter. 

![](https://s3.amazonaws.com/images.willricketts.com/rhea/astrosynthesis.png)

Before long, I had a galaxy containing ~20 solar systems and connections between them, which I exported to a large XML document, which I was able to load into the Elixir-based backend of Rhea with [elixir-XML-to-map](https://github.com/homanchou/elixir-xml-to-map).


#### Using the Data

From this _very_ large map, I perform a series of mutations on the data to order it into a list of Solar Systems and one of Gates. With these lists, I first created a DB record for each of the solar systems within PostgreSQL. With a SolarSystems table populated with data, I would need to represent the data somewhere that graph-oriented data feels right at home. This is where [Neo4j](https://neo4j.com/) comes into play.

If you're unfamiliar, Neo4j is a graph database. This turned out to be the perfect tool for Rhea. It was immediately apparent that its Cypher query language had the features needed to power several in-game systems for Rhea. Namely its ability to find the shortest path between nodes, or in the domain of Rhea, solar systems.

Claiming victory over my first set of development goals, I took a 2-month break from working on Rhea to focus on a large project I was leading at work, a large web service responsible for creating and marshaling Twilio conference calls in parallel-- also built using an Elixir umbrella, also structured with the intent of utilizing BEAM's distribution features in the future.

When I finally had time to devote to Rhea again, I revisited the map I'd generated and simply wasn't happy with the result. After generating entirely new map data, this time for 2000 solar systems, I was dismayed to learn that the XML schema for my new map data was fundamentally different than that of my previous map. This would require me to write an entirely new parser and process for loading the map data into PostgreSQL and Neo4j. This was unacceptable to me.

Not being comfortable with the idea of such a pivotal piece of the system relying upon a proprietary dependency, I started exploring ways of generating my own map from scratch, but how?

#### Enter Diffusion Limited Aggregation

A while back, I was reading about [Brownian Motion](https://en.wikipedia.org/wiki/Brownian_motion) and the aggregation of particles to form a structure. After diving back down that rabbit hole, I discovered [Diffusion Limited Aggregation](https://en.wikipedia.org/wiki/Diffusion-limited_aggregation). This seemed to be the ideal method for generating a structure akin to the galactic maps of both Stellaris and Eve Online.

I found a simple [C++ implementation of DLA](https://github.com/fogleman/dlaf) and quickly generated a set of 2000 particles. This algorithm has a CSV output of the following schema:

```
id, parent_id, x, y, z
0,-1,0,0,0
1,0,0.934937,0.354814,0
2,0,0.0525095,-0.99862,0
3,1,0.989836,1.35331,0
4,3,1.92472,1.70826,0
5,3,0.65572,2.29584,0
...
```

<br />

<iframe width="560" height="315" src="https://www.youtube.com/embed/EWks4tbH9b0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> 

EUREKA!

That was precisely what I needed to generate a fully original map! I built out each point as a Solar System in Postgres and Neo4j with its coordinates. After reducing the list down to a collection of IDs and Parent IDs, I was then able to build the stargates that connect the various solar systems together.

Originally, [I built a name generator to automatically name each Solar System](https://gist.github.com/willricketts/dfc4e2ce63d8a9934434d18cdb860510), but this was quickly outgrown and I was once again searching for a solution to unblock my map work.

After finding a [super slick name generator built in Javascript](https://github.com/hbi99/namegen/blob/master/namegen.js), I was able to give each solar system a unique name that doesn't sound like a Heroku deployment.


Below: the final structure of my fractally generated galaxy map:

![](https://s3.amazonaws.com/images.willricketts.com/rhea/rheamapsmall.png)

Below: a closer view of a group of solar system:

![](https://s3.amazonaws.com/images.willricketts.com/rhea/rheamapclose.png)

### Future Plans

Having a totally original galaxy map with a predictable and repeatable process for generation opens up all sorts of possibilities for new features and clever tricks. Having tied all of the generation and persistence process to a single function in the backend, rebuilding the map and trying generation with different inputs takes a matter of seconds::

```elixir
def build_universe do
  construct_map_db()
  construct_graph()

  IO.puts "Universe created"
end

# I couldn't resist writing a function called "destroy universe"
def destroy_universe do
  [{:ok, _}, _] = [
    Task.async(&Main.destroy_map_db/0),
    Task.async(&Main.destroy_graph/0)
  ] 
  |> Enum.map(&Task.await/1)

  IO.puts "Universe destroyed"
end
```

#### Wormholes

If you're familiar with graphs, then you were probably left wondering why I refer to Rhea's map as a _directed_ acyclic graph as opposed to an _undirected_ acyclic graph. Rhea's stargates are represented in its persistence layer as pairs of gates, each unidirectional. This enables me to, at some point, build a wormhole system in which a player's fleets can travel from one part of the galaxy to another instantly, but perhaps not return home through the same wormhole.

#### Tactical Structures

Perhaps I'll introduce a player owned structure that prevents enemy fleets from leaving a Solar System once they've traveled through a stargate. This would lead to a much more robust system of player combat tactics and provide an overall more engaging experience, possibly allowing the few to overcome the many in a fleet engagement.

### Next Steps

The map is far too linear, and this needs to be fixed before proceeding. This is my primary development item on the project, and I'm currently working on an algorithm that utilizes Solar System coordinates to create connections between solar systems that share a geographic region with one another. I assure you that will get its own blog post :)

For now, I'm happy enough with my work to begin sharing the obstacles I've overcome in building the game I've always wanted.

Thanks for reading, and I'll see you in the next post.