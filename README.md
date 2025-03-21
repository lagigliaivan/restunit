# restunit

This library allows you to run http requests as part of a unit test. 

# example

```golang
POSTJob := restunit.NewRequestWithResponse[any, model.PostResponse](cfg.router)
POSTJob.
  URL("/myurl/resource").
  POST(postRequest).
  Expect(http.StatusCreated)

h := map[string]string{"Authorization": "Bearer " + "token"}

GETJobs := restunit.NewRequestWithResponse[model.Reqest,model.Response](cfg.router)
response := GETJobs.URL(fmt.Sprintf("/myurl/resource/items?now=%s", now)).
      WithHeaders(h).
      GET().
      Expect(http.StatusOK).
      Body()

assert.NotEmpty(t, response.Items)
```
