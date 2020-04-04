## 执行计划

#### 例
执行
```
db.user.find({tags:"music"}).explain()
```
返回
```
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "mongotest.user",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "tags" : {
                "$eq" : "music"
            }
        },
        "winningPlan" : {
            "stage" : "FETCH",
            "inputStage" : {
                "stage" : "IXSCAN",
                "keyPattern" : {
                    "tags" : 1.0
                },
                "indexName" : "tags_1",
                "isMultiKey" : true,
                "multiKeyPaths" : {
                    "tags" : [
                        "tags"
                    ]
                },
                "isUnique" : false,
                "isSparse" : false,
                "isPartial" : false,
                "indexVersion" : 2,
                "direction" : "forward",
                "indexBounds" : {
                    "tags" : [
                        "[\"music\", \"music\"]"
                    ]
                }
            }
        },
        "rejectedPlans" : []
    },
    "serverInfo" : {
        "host" : "kf-PC",
        "port" : 27017,
        "version" : "3.4.9",
        "gitVersion" : "876ebee8c7dd0e2d992f36a848ff4dc50ee6603e"
    },
    "ok" : 1.0
}
```

参考文档：
1. https://docs.mongodb.org/v3.0/reference/explain-results/
2. https://blog.csdn.net/fly910905/article/details/78336961