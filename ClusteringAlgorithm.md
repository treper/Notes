###Clustering Algorithm
####DBSCAN
```
DBSCAN(D, eps, MinPts)
   C = 0
   for each unvisited point P in dataset D
      mark P as visited
      NeighborPts = regionQuery(P, eps)
      if sizeof(NeighborPts) < MinPts
         mark P as NOISE
      else
         C = next cluster
         expandCluster(P, NeighborPts, C, eps, MinPts)
          
expandCluster(P, NeighborPts, C, eps, MinPts)
   add P to cluster C
   for each point P' in NeighborPts 
      if P' is not visited
         mark P' as visited
         NeighborPts' = regionQuery(P', eps)
         if sizeof(NeighborPts') >= MinPts
            NeighborPts = NeighborPts joined with NeighborPts'
      if P' is not yet member of any cluster
         add P' to cluster C
          
regionQuery(P, eps)
   return all points within P's eps-neighborhood (including P)
```

####OPTICS

The basic approach of OPTICS is similar to DBSCAN, but instead of maintaining a set of known, but so far unprocessed cluster members, a priority queue (e.g. using an indexed heap) is used.
```
 OPTICS(DB, eps, MinPts)
    for each point p of DB
       p.reachability-distance = UNDEFINED
    for each unprocessed point p of DB
       N = getNeighbors(p, eps)
       mark p as processed
       output p to the ordered list
       Seeds = empty priority queue
       if (core-distance(p, eps, Minpts) != UNDEFINED)
          update(N, p, Seeds, eps, Minpts)
          for each next q in Seeds
             N' = getNeighbors(q, eps)
             mark q as processed
             output q to the ordered list
             if (core-distance(q, eps, Minpts) != UNDEFINED)
                update(N', q, Seeds, eps, Minpts)
```
In update(), the priority queue Seeds is updated with the \varepsilon-neighborhood of p and q, respectively:
```
 update(N, p, Seeds, eps, Minpts)
    coredist = core-distance(p, eps, MinPts)
    for each o in N
       if (o is not processed)
          new-reach-dist = max(coredist, dist(p,o))
          if (o.reachability-distance == UNDEFINED) // o is not in Seeds
              o.reachability-distance = new-reach-dist
              Seeds.insert(o, new-reach-dist)
          else               // o in Seeds, check for improvement
              if (new-reach-dist < o.reachability-distance)
                 o.reachability-distance = new-reach-dist
                 Seeds.move-up(o, new-reach-dist)

OPTICS hence outputs the points in a particular ordering, annotated with their smallest reachability distance (in the original algorithm, the core distance is also exported, but this is not required for further processing).
```