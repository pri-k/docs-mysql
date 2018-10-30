docs-mysql
==========

Docs for MySQL for [Pivotal Cloud Foundry](https://network.pivotal.io/products/pivotal-cf) (PCF)

## Which branch to use?

**Note**: Provide instructions in your PRs to indicate which branches you want Docs to apply your commits to. 

| Branch name | Use for… |
|-------------| ------|
| master      | "edge" branch for 2.x, publishes to https://docs-pcf-staging.cfapps.io/p-mysql/2-n/|
| 2.4         | v2.4.x | 
| 2.3         | v2.3.x | 
| 2.2         | v2.2.x |  
| 2.1         | v2.1.x — Now PDFed: https://docs.pivotal.io/archives/mysql-docs-2.1.pdf | 
| 2.0         | v2.0.x — Now PDFed: https://docs.pivotal.io/archives/mysql-docs-2.0.pdf |
| 1.11        | There are no plans for a v1.11. However, because it publishes to the edge branch, you could use it to stage big changes for the v1.10.x without worrying they'll go to production prematurely. |
| 1.10        | v1.10.x |
| 1.9         | v1.9.x — Now PDFed: https://docs.pivotal.io/archives/mysql-docs-1.9.pdf |

**Note**: Branches v1.9 through v1.4 and v2.0 are no longer published as live documentation. However, documentation for those versions of PDFs is available as PDFs.

## Partials

Cross-product partials for **MySQL for PCF** are single sourced from the [Services Partials](https://github.com/pivotal-cf/docs-services-partials) repo to allow for versioning.

Previously, these partials were sourced from the v018.x branch of the [On Demand Service Broker SDK](https://github.com/pivotal-cf/docs-on-demand-service-broker/tree/v0.18.x) repo and did not support versions.

## Steps for local development
```
$ git clone git@github.com:pivotal-cf/docs-layout-repo 
$ git clone git@github.com:pivotal-cf/docs-mysql
$ cd docs-mysql && git checkout <branch> && cd -
$ git clone git@github.com:pivotal-cf/docs-book-mysql
$ cd docs-book-mysql
$ bundle install
$ bundle exec bookbinder watch
$ open http://127.0.0.1:XXXX/p-mysql/<branch>
```
