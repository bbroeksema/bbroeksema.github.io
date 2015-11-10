# Cluster validation

In many contexts the need arises for segmenting a dataset. Preferably, this is
done automatically, but given the wide range of available clustering algorithms
it should be clear that no one-fits-all approach exists. As a consequence,
research was done over the past three decades to find measures of "goodness" or
cluster validity indices (CVI). A recent (2013) paper compares thirthy (30) of
them, again pointing out a lack of agreement of what gives the best results. And
frankly, best results pretty much depend on both the properties of the data set
that is being clustered, the algorithm chosen for clustering, the parametrization
of the algorithm and the used CVI.

This is a major problem for end-users, that is users which work in a particular
context (e.g. biology, finance or healthcare). End-users who might have some
understanding of differences between clustering algorithms and/or CVIs, but for
sure will not have a deep knowledge about all the intricate details that come to
play. Let alone, that they understand the interplay between clustering algorithm,
algorithm configuration and CVI.

As a consequence, there is a lot of research done on each of these topics, in
the sense of new clustering algorithms, new CVIs and comparative studies.
However, the result seems to be an enormous search space for which no tools or
best practices exists, which *conveniently* support end-users with their tasks
of segmenting the data. And one should keep in mind that clustering is not an
end, but a means to an end for these users.
