# api documentation for  [mongoose-paginate (v5.0.3)](https://github.com/edwardhotchkiss/mongoose-paginate#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-mongoose-paginate.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-mongoose-paginate) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-mongoose-paginate.svg)](https://travis-ci.org/npmdoc/node-npmdoc-mongoose-paginate)
#### Pagination plugin for Mongoose

[![NPM](https://nodei.co/npm/mongoose-paginate.png?downloads=true)](https://www.npmjs.com/package/mongoose-paginate)

[![apidoc](https://npmdoc.github.io/node-npmdoc-mongoose-paginate/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-mongoose-paginate_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-mongoose-paginate/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-mongoose-paginate/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-mongoose-paginate/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Edward Hotchkiss",
        "email": "edward@edwardhotchkiss.com"
    },
    "bugs": {
        "url": "https://github.com/edwardhotchkiss/mongoose-paginate/issues"
    },
    "contributors": [
        {
            "name": "Edward Hotchkiss",
            "email": "edward@edwardhotchkiss.com"
        },
        {
            "name": "Nick Baugh",
            "email": "niftylettuce@gmail.com"
        },
        {
            "name": "Dmitry Kirilyuk",
            "email": "gk.joker@gmail.com"
        }
    ],
    "dependencies": {
        "bluebird": "3.0.5"
    },
    "description": "Pagination plugin for Mongoose",
    "devDependencies": {
        "chai": "3.4.1",
        "chai-as-promised": "5.1.0",
        "mocha": "2.3.4",
        "mongoose": "4.2.8"
    },
    "directories": {},
    "dist": {
        "shasum": "d7ae49ed5bf64f1f7af7620ea865b67058c55371",
        "tarball": "https://registry.npmjs.org/mongoose-paginate/-/mongoose-paginate-5.0.3.tgz"
    },
    "engines": {
        "node": ">=4.0.0"
    },
    "gitHead": "9a4d3612a589cbe267eb7b09f9edd4460181d219",
    "homepage": "https://github.com/edwardhotchkiss/mongoose-paginate#readme",
    "keywords": [
        "mongoose",
        "paginate",
        "pagination",
        "paging",
        "page"
    ],
    "license": "MIT",
    "maintainers": [
        {
            "name": "edwardhotchkiss",
            "email": "edward@edwardhotchkiss.com"
        },
        {
            "name": "jokero",
            "email": "gk.joker@gmail.com"
        },
        {
            "name": "niftylettuce",
            "email": "niftylettuce@gmail.com"
        }
    ],
    "name": "mongoose-paginate",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/edwardhotchkiss/mongoose-paginate.git"
    },
    "scripts": {
        "test": "mocha tests/*.js --timeout 5000"
    },
    "version": "5.0.3"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module mongoose-paginate](#apidoc.module.mongoose-paginate)
1.  [function <span class="apidocSignatureSpan">mongoose-paginate.</span>paginate (query, options, callback)](#apidoc.element.mongoose-paginate.paginate)



# <a name="apidoc.module.mongoose-paginate"></a>[module mongoose-paginate](#apidoc.module.mongoose-paginate)

#### <a name="apidoc.element.mongoose-paginate.paginate"></a>[function <span class="apidocSignatureSpan">mongoose-paginate.</span>paginate (query, options, callback)](#apidoc.element.mongoose-paginate.paginate)
- description and source-code
```javascript
function paginate(query, options, callback) {
    query   = query || {};
    options = Object.assign({}, paginate.options, options);

    var select     = options.select;
    var sort       = options.sort;
    var populate   = options.populate;
    var lean       = options.lean || false;
    var leanWithId = options.hasOwnProperty('leanWithId') ? options.leanWithId : true;

    var limit = options.hasOwnProperty('limit') ? options.limit : 10;
    var skip, offset, page;

    if (options.hasOwnProperty('offset')) {
        offset = options.offset;
        skip   = offset;
    } else if (options.hasOwnProperty('page')) {
        page = options.page;
        skip = (page - 1) * limit;
    } else {
        offset = 0;
        page   = 1;
        skip   = offset;
    }

    var promises = {
        docs:  Promise.resolve([]),
        count: this.count(query).exec()
    };

    if (limit) {
        var query = this.find(query)
                        .select(select)
                        .sort(sort)
                        .skip(skip)
                        .limit(limit)
                        .lean(lean);

        if (populate) {
            [].concat(populate).forEach(function(item) {
                query.populate(item);
            });
        }

        promises.docs = query.exec();

        if (lean && leanWithId) {
            promises.docs = promises.docs.then(function(docs) {
                docs.forEach(function(doc) {
                    doc.id = String(doc._id);
                });

                return docs;
            });
        }
    }

    return Promise.props(promises)
        .then(function(data) {
            var result = {
                docs:  data.docs,
                total: data.count,
                limit: limit
            };

            if (offset !== undefined) {
                result.offset = offset;
            }

            if (page !== undefined) {
                result.page  = page;
                result.pages = Math.ceil(data.count / limit) || 1;
            }

            return result;
        })
        .asCallback(callback);
}
```
- example usage
```shell
...
'''js
var mongoose         = require('mongoose');
var mongoosePaginate = require('mongoose-paginate');

var schema = new mongoose.Schema({ /* schema definition */ });
schema.plugin(mongoosePaginate);

var Model = mongoose.model('Model',  schema); // Model.paginate()
'''

### Model.paginate([query], [options], [callback])

Returns promise

**Parameters**
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
