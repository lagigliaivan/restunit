# restunit

This library allows you to run http requests as part of a unit test. 

# example

```golang
func TestDeleteOutOfDateAccounts(t *testing.T) {
	testharness.New().WithBasicSetup(func(app *dig.Container) {
		service.ProvideServices(app)
		handler.ProvideHandlers(app)

		c := CreateTestConfig(app)

		body := common.BulkUpsertBody[model.GlAccount]{
			Data: getGLAccounts(),
		}

		//POSTing bankaccount
		p := juez.NewRequestWithResponse[common.BulkUpsertBody[model.GlAccount], []model.GlAccount](c.router).
			WithHeaders(c.authHeader).
			URL("/api/v1/bank_accounts").
			POST(body).
			Expect(http.StatusOK).
			Body()

		assert.Len(t, p, 4)

		body = common.BulkUpsertBody[model.GlAccount]{
			Data: []model.GlAccount{getGLAccounts()[0]},
		}

		time.Sleep(1 * time.Second) //Sleeping to make sure the updated_at is different

		//UPDATing bankaccount
		p = juez.NewRequestWithResponse[common.BulkUpsertBody[model.GlAccount], []model.GlAccount](c.router).
			WithHeaders(c.authHeader).
			URL("/api/v1/bank_accounts").
			POST(body).
			Expect(http.StatusOK).
			Body()

		//DELETEing property
		juez.NewRequest[any](c.router).
			WithHeaders(c.authHeader).
			URL(fmt.Sprintf("/api/v1/bank_accounts?until=%s&customer_id=%s",
				p[0].UpdatedAt.Format(time.RFC3339),
				body.Data[0].CustomerId,
			)).
			DELETE().
			Expect(http.StatusOK)

		//GETting property
		response := juez.NewRequestWithResponse[any, common.PaginatedResponse[model.GlAccount]](c.router).
			WithHeaders(c.authHeader).
			URL("/api/v1/bank_accounts?filter[customer_id]=a25aae91-475d-11ee-8201-0a58a9feac02").
			GET().
			Expect(http.StatusOK).Body()

		assert.Len(t, response.Items, 1)

		var accounts []model.BankAccount
		testharness.NewMigratedTestDB().DB.Unscoped().Where("customer_id = ?", body.Data[0].CustomerId).Find(&accounts)
		assert.Len(t, accounts, 3)

		//GETting property
		response = juez.NewRequestWithResponse[any, common.PaginatedResponse[model.GlAccount]](c.router).
			WithHeaders(c.authHeader).
			URL("/api/v1/bank_accounts?filter[customer_id]=a25aae91-475d-11ee-8201-0a58a9feac01").
			GET().
			Expect(http.StatusOK).Body()

		assert.Len(t, response.Items, 1)

		testharness.NewMigratedTestDB().DB.Unscoped().Where("customer_id = ?", "a25aae91-475d-11ee-8201-0a58a9feac01").
			Find(&accounts)

		assert.Len(t, accounts, 1)
	})
}
```
