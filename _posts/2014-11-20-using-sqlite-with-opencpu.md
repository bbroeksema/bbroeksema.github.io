---
title: Using SQLite with OpenCPU
layout: post
tags: r sqlite opencpu development
---

In a project I am currently working on, we develop an R-package with a web interface for interactive data exploration and clustering.
To make the coupling between our R code and the web frontend, we use [OpenCPU](https://www.opencpu.org/).
For performance reasons, we decided at some point to store and update data in an SQLite database.
Using SQLite in R is pretty straight forward, thanks to the [RSQLite](http://cran.r-project.org/web/packages/RSQLite/index.html) package.
Making it work with OpenCPU was a little bit less straight forward, though we managed in the end.

In this post I will describe the hoops we went through to make it work.
A cautionary note: I describe how we made this work, which is not the same as best practice.
In our case the database basically serves as a cache, and as such collides with OpenCPU's caching method, which I will explain in more detail below.

<br>
**Assumptions:** Reasonable knowledge on how OpenCPU works.
Familiar with R, R programming and R package development.
Linux is assumed to be the OS at hand (some of these issue might not appear on Windows, or require different solutions. I have not tested this).

# Some background

The overall context of this prototype pertains to the problem of contig binning in metagenomics.
Some more information about our approach can be found in a [poster (pdf)](https://www.researchgate.net/profile/Mohammad_Ghoniem/publication/265064825_Interactive_Visual_Support_for_Metagenomic_Contig_Binning/links/5448d2490cf2f14fb81447d2?ev=pub_int_doc_dl&origin=publication_detail&inViewer=true), which we recently published at VIS 2014, in Paris.
A brief video of an early prototype is available [here (vimeo)](http://vimeo.com/103146392).

In our approach, we have a number of dynamic columns.
That is, columns which are calculated based on values in other columns.
Some examples: principle axes obtained using PCA, and cluster ids obtained using a variety of clustering methods.
We want everything in the same (logical) table, because some dynamic columns might be input for others.
For example, we might want to include principal components in one of our clustering methods.

This leads to performance issues when using OpenCPU for the following reason.
As illustrated, some operation lead to a new column, or to updated values of an existing column.
To make this happen in OpenCPU, the function that is adding the column, needs to load the full data table.
Next, it needs to extract the columns on which the operation is to be performed.
Finally, it adds the resulting column to the full table or updates an existing one.
In an R session this looks more or less like:

    > table <- data(mtcars)
    > library(FactoMineR)
    > pca.res <- PCA(table[c("mpg", "cyl", "disp")])
    > table$pca1 <- as.vector(pca.res$ind$coord[,1])
    > table

To make this work with OpenCPU, above code is wrapped into a function, which can next be called from the front end.
Additionally, the table is not loaded in the function, but passed as a parameter.
This way, an OpenCPU session can be passed in for successive calls of this function.

Now, OpenCPU is designed in such a way, that it stores the full session.
That is, it stores the function parameters (e.g. the columns on which to perform PCA) and the final result (the modified table).
To reuse the results of our PCA, we keep the OpenCPU session object in the frontend, and pass it to our clustering function.

So, the design issue of OpenCPU, that caused a performance bottleneck for us, is the fact that the full table must be loaded and stored for each function call.
Even if we update an existing column, still, the full table will be written in the session object.
We have went through various design iterations to cope with this, which I hope to detail in another blog post.
At this stage, we settled with using an SQLite database to work around these issues.

# Using SQLite

The idea is simple, in stead of returning the modified table from our functions, we update an SQLite database.
One of the advantages is that SQLite is pretty light-weight, so we still can have a simple to install R-package.
Simple as in, it does not require the user to setup a database system, which would be painful for our target audience (biologists).

The first step is to move our data into an SQLite database.
This can be done easily using the RSQLite package:

    db.init <- function() {
      data(mtcars)

      if (!file.exists("/path/to/database.db")) {
        dir.create("/path/to/database.db, recursive = T)
      }

      con <- DBI::dbConnect(RSQLite::SQLite(), "/path/to/database.db")
      DBI::dbWriteTable(con, "mtcars", mtcars, row.names=F)
      DBI::dbDisconnect(con)
    }

This code, works perfectly fine in R.
However, when calling this same function through OpenCPU, things will go wrong.
Though, its documentation gives some information about the interplay with AppArmor due to its security model, it took a bit to figure out what was to be done in this particular case.

The important thing to keep in mind here is that SQLite is a one-file database.
The other important thing to know is that the file will be read/written by www-data. 
As such, we need to instruct AppArmor to give read/write access to the appropriate directory, as well as read write access to the database file.
Let's assume that the package is called MyPackage, and that the db file is stored in /tmp/MyPackage/cache.db.
Then, configuring [AppArmor](http://manpages.ubuntu.com/manpages/intrepid/man5/apparmor.d.5.html) is done by adding the following two lines to /etc/apparmor.d/opencpu.d/custom:

    /tmp/MyPackage/ rw,
    /tmp/MyPackage/** rwkmix,

# The caching problem

OpenCPU has a very nice caching feature, which can considerably speed up consecutive calls to the same function.
The mechanism is based on the assumption that all functions, provided by a package, are pure functions.
That is, none of the functions have any side effects.
Pure functions will always return exactly the same results, when called with the same parameters.

However, this is problematic when functions are **not** pure and thus have side effects.
Let's say we have a function columns, it has no arguments and returns a list of column names.
Now, each function that adds a column to the database (e.g. cluster(), which adds a new column with cluster ids), will break the column function.
If columns is called before cluster is called, the results will be cached by OpenCPU.
When it is called again, after cluster has being called, it will return the cached results from the earlier call.
As such, the return value will not contain the newly added column.
To overcome this problem, the opencpu-cache service must be stopped:

    sudo service opencpu-cache stop

Clearly, this comes at the price of having cached results, but these where part of my initial problem in the first place.
As such, this approach can be seen as a trade-of between making high-volume actions more efficient versus making highly repetitive actions more efficient.

# Conclusion

To summarize, it is possible to use SQLite in an OpenCPU based application.
However, it requires both proper configuration of AppArmor, and disabling opencpu cache.
Especially, the latter is a sign of breaking the pure functional approach assumption that underlies OpenCPU.
It is not clear to me yet if I am trying to use OpenCPU for something that it was not designed for, or that there are other ways of dealing with the performance issues I tried to cope with.
