+++
title = "go-zero 单体应用实践（三）"
description = "单体应用实践"
weight = 3
url = "/go-zero/03.html"
+++
# 列表数据分页
1、api 定义 Pager api 可以单独一个文件；此时 userlogin.api 需要  import "api/pager.api"
```go
Pager {
    Page int `form:"page,default=1"`
    PageSize int `form:"page_size,default=10"`
    TotalSize int `form:"total_size,default=0"`
}

TagListRequest {
		Name string `form:"name,optional"`
		Pager
}
TagListResponse {
    List     []TagResponse `json:"list"`
    Matedata Pager         `json:"matedata"`
}


service userlogin-api {
	@handler Tags
	get /api/tag/list(TagListRequest) returns (TagListResponse)
}
```

2、model 文件：BlogTagModel增加TageList方法；同时 customBlogTagModel.TagList 实现具体逻辑
```go
BlogTagModel interface {
		blogTagModel
		TagList(context.Context, *types.TagListRequest) (*types.TagListResponse, error)
}


func (c *customBlogTagModel) TagList(ctx context.Context, request *types.TagListRequest) (*types.TagListResponse, error) {
	result := &types.TagListResponse{}
	pager := request.Pager
	countSql := fmt.Sprintf("select %s from %s where 1 ", "count(id) as total_size", c.table)
	_ = c.QueryRowNoCacheCtx(ctx, &pager.TotalSize, countSql)
	result.Matedata = pager
	where := "1 "
	var err error
	if len(request.Name) > 0 {
		where += "AND name=? "
		sql := fmt.Sprintf("select %s from %s where %s  limit %d,%d",
			blogTagRows, c.table, where, (pager.Page-1)*pager.PageSize, request.PageSize)

		err = c.QueryRowsNoCacheCtx(ctx, &result.List, sql, request.Name)
	} else {
		sql := fmt.Sprintf("select %s from %s where %s  limit %d,%d",
			blogTagRows, c.table, where, (pager.Page-1)*pager.PageSize, request.PageSize)

		err = c.QueryRowsNoCacheCtx(ctx, &result.List, sql)
	}
	switch err {
	case nil:
		return result, nil
	case sqlc.ErrNotFound:
		return nil, ErrNotFound
	default:
		return nil, err
	}
}
```

3、logic 文件编辑
```go
func (l *TagsLogic) Tags(req *types.TagListRequest) (resp *types.TagListResponse, err error) {
	return l.svcCtx.TagsModel.TagList(l.ctx, req)
}
```
4、测试  [http://127.0.0.1:8000/api/tag/list?page=2&page_size=5](http://127.0.0.1:8000/api/tag/list?page=2&page_size=5)
```json
{
    "list": [
        {
            "id": 6,
            "name": "python",
            "state": 1
        },
        {
            "id": 7,
            "name": "rust",
            "state": 1
        },
        {
            "id": 8,
            "name": "typescript",
            "state": 1
        }
    ],
    "matedata": {
        "Page": 2,
        "PageSize": 5,
        "TotalSize": 8
    }
}
```

