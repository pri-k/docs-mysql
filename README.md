docs-mysql
==========

Docs for MySQL for [Pivotal Cloud Foundry](https://network.pivotal.io/products/pivotal-cf) (PCF)

## Which branch to use?

**Note**: Provide instructions in your PRs to indicate which branches you want Docs to apply your commits to. 

| Branch name | Use for… |
|-------------| ------|
| master      | NOT IN USE|
| 2.3         | "edge" build | 
| 2.2         | v2.2.x | 
| 2.1         | v2.1.x | 
| 2.0         | v2.0.x |
| 1.11        | There are no plans for a v1.11. However, because it publishes to the edge branch, you could use it to stage big changes for the v1.10.x without worrying they'll go to production prematurely. |
| 1.10        | v1.10.x |
| 1.9         | v1.9.x |
| 1.8         | v1.8.x |
| 1.7         | v1.7.x |

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
