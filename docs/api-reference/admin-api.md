---
sidebar_position: 5
description: "Administrative endpoints for managing accounts, bets, and wagers."
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Admin APIs

## Get Single Account

This endpoint allows administrators to retrieve details for a specific account.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                          |
| :----------- | :----------------------------- |
| method       | `GET`                          |
| url          | `$baseUrl/admin/accounts/:id`  |
| Content-Type | `application/json`             |

#### Path Parameters

| Parameter | Description              |
| --------- | ------------------------ |
| id        | The ID of the account    |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/admin/accounts/2" \
      -H "Authorization: Bearer $token"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const getAccount = async (accountId) => {
      try {
        const response = await fetch(`${baseUrl}/admin/accounts/${accountId}`, {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });
        
        const data = await response.json();
        console.log('Account details:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const accountId = 2;
    getAccount(accountId);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_account(token, account_id):
        url = f"{base_url}/admin/accounts/{account_id}"
        
        headers = {
            'Authorization': f'Bearer {token}'
        }
        
        response = requests.get(url, headers=headers)

        if response.status_code == 200:
            print('Account details:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    account_id = 2

    account = get_account(token, account_id)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::error::Error;

    async fn get_account(
        client: &Client, 
        base_url: &str, 
        token: &str,
        account_id: u64
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/accounts/{}", base_url, account_id);
        
        let response = client.get(&url)
            .header("Authorization", format!("Bearer {}", token))
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Account details: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let base_url = "your_base_url"; // Replace with actual base URL
        let token = "your_auth_token"; // Replace with actual token
        let account_id = 2;

        get_account(&client, base_url, token, account_id).await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem value="Success">

    Http Code: `200`
    ```json
    {
      "data": {
        "id": 2,
        "name": "First Account",
        "email": "first@email.com",
        "email_verified_at": "2025-05-08T23:16:29.000Z",
        "api_key_generated_at": "2025-05-08T23:16:29.000Z",
        "preferences": {
          "allow_negative_balance": false
        },
        "created_at": "2025-05-08T15:21:41.017Z",
        "updated_at": "2025-05-08T23:17:25.456Z"
      },
      "message": "Account retrieved successfully"
    }
    ```

  </TabItem>
  <TabItem value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **Forbidden** <br/>
    Http Code: `403`
    ```json
    {
      "status": 403,
      "error": "Admin access required"
    }
    ```

    **Not Found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "Account not found"
    }
    ```

  </TabItem>
</Tabs>

## Get Paginated Accounts

This endpoint allows administrators to retrieve a paginated list of all accounts.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                       |
| :----------- | :-------------------------- |
| method       | `GET`                       |
| url          | `$baseUrl/admin/accounts`   |
| Content-Type | `application/json`          |

#### Query Parameters

| Parameter   | Type   | Required | Default | Description                       |
| ----------- | ------ | -------- | ------- | --------------------------------- |
| page        | number | No       | 1       | Page number for pagination        |
| per_page    | number | No       | 20      | Number of results per page        |
| search      | string | No       | -       | Search term for account name/email|

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/admin/accounts?page=1&per_page=10&search=email" \
      -H "Authorization: Bearer $token"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const getAccounts = async (options = {}) => {
      try {
        const params = new URLSearchParams();
        
        if (options.page) params.append('page', options.page);
        if (options.perPage) params.append('per_page', options.perPage);
        if (options.search) params.append('search', options.search);
        
        const response = await fetch(`${baseUrl}/admin/accounts?${params.toString()}`, {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });
        
        const data = await response.json();
        console.log('Accounts:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const options = {
      page: 1,
      perPage: 10,
      search: 'email'
    };

    getAccounts(options);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    from urllib.parse import urlencode

    def get_accounts(token, page=1, per_page=20, search=None):
        params = {
            'page': page,
            'per_page': per_page
        }
        
        if search:
            params['search'] = search
            
        query_string = urlencode(params)
        url = f"{base_url}/admin/accounts?{query_string}"
        
        headers = {
            'Authorization': f'Bearer {token}'
        }
        
        response = requests.get(url, headers=headers)

        if response.status_code == 200:
            print('Accounts:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token

    accounts = get_accounts(token, page=1, per_page=10, search='email')
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::error::Error;

    async fn get_accounts(
        client: &Client, 
        base_url: &str, 
        token: &str,
        page: Option<u32>,
        per_page: Option<u32>,
        search: Option<&str>
    ) -> Result<(), Box<dyn Error>> {
        let mut url = format!("{}/admin/accounts?", base_url);
        
        // Add query parameters
        if let Some(p) = page {
            url.push_str(&format!("page={}&", p));
        }
        
        if let Some(pp) = per_page {
            url.push_str(&format!("per_page={}&", pp));
        }
        
        if let Some(s) = search {
            url.push_str(&format!("search={}&", s));
        }
        
        // Remove trailing & if exists
        if url.ends_with('&') {
            url.pop();
        }
        
        let response = client.get(&url)
            .header("Authorization", format!("Bearer {}", token))
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Accounts: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let base_url = "your_base_url"; // Replace with actual base URL
        let token = "your_auth_token"; // Replace with actual token
        
        get_accounts(
            &client, 
            base_url, 
            token, 
            Some(1), 
            Some(10), 
            Some("email")
        ).await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem value="Success">

    Http Code: `200`
    ```json
    {
      "data": [
        {
          "id": 2,
          "name": "First Account",
          "email": "first@email.com",
          "email_verified_at": "2025-05-08T23:16:29.000Z",
          "api_key_generated_at": "2025-05-08T23:16:29.000Z",
          "preferences": {
            "allow_negative_balance": false
          },
          "created_at": "2025-05-08T15:21:41.017Z",
          "updated_at": "2025-05-08T23:17:25.456Z"
        },
        {
          "id": 1,
          "name": "Admin Account",
          "email": "admin@email.com",
          "email_verified_at": "2025-05-01T12:00:00.000Z",
          "api_key_generated_at": "2025-05-01T12:00:00.000Z",
          "preferences": {},
          "created_at": "2025-05-01T12:00:00.000Z",
          "updated_at": "2025-05-01T12:00:00.000Z"
        }
      ],
      "per_page": 20,
      "page": 1,
      "total": 2,
      "from": 1,
      "to": 2,
      "last_page": 1
    }
    ```

  </TabItem>
  <TabItem value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **Forbidden** <br/>
    Http Code: `403`
    ```json
    {
      "status": 403,
      "error": "Admin access required"
    }
    ```

  </TabItem>
</Tabs>

## Rollback Bet

This endpoint allows administrators to rollback a bet to its previous state, reversing all financial transactions associated with it.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                |
| :----------- | :----------------------------------- |
| method       | `POST`                               |
| url          | `$baseUrl/admin/bets/:betId/rollback` |
| Content-Type | `application/json`                   |

#### Path Parameters

| Parameter | Description          |
| --------- | -------------------- |
| betId     | The ID of the bet    |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/admin/bets/7/rollback" \
      -H "Authorization: Bearer $token"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const rollbackBet = async (betId) => {
      try {
        const response = await fetch(`${baseUrl}/admin/bets/${betId}/rollback`, {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });
        
        const data = await response.json();
        console.log('Rollback result:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const betId = 7;
    rollbackBet(betId);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def rollback_bet(token, bet_id):
        url = f"{base_url}/admin/bets/{bet_id}/rollback"
        
        headers = {
            'Authorization': f'Bearer {token}'
        }
        
        response = requests.post(url, headers=headers)

        if response.status_code == 200:
            print('Rollback successful:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    bet_id = 7

    result = rollback_bet(token, bet_id)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::error::Error;

    async fn rollback_bet(
        client: &Client, 
        base_url: &str, 
        token: &str,
        bet_id: u64
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/bets/{}/rollback", base_url, bet_id);
        
        let response = client.post(&url)
            .header("Authorization", format!("Bearer {}", token))
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Rollback successful: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let base_url = "your_base_url"; // Replace with actual base URL
        let token = "your_auth_token"; // Replace with actual token
        let bet_id = 7;

        rollback_bet(&client, base_url, token, bet_id).await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem value="Success">

    Http Code: `200`
    ```json
    {
      "message": "Bet rolled back successfully"
    }
    ```

  </TabItem>
  <TabItem value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **Forbidden** <br/>
    Http Code: `403`
    ```json
    {
      "status": 403,
      "error": "Admin access required"
    }
    ```

    **Already Rolled Back** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bet already rolled back"
    }
    ```

    **Not Found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "Bet not found"
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to rollback bet",
      "data": [
        {
          "message": "User balance not sufficient to rollback bet",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_rollback": -90000,
            "current_exposure": 0,
            "exposure_after_rollback": 0
          },
          "associated_bet_ids": [
            7
          ]
        }
      ]
    }
    ```

  </TabItem>
</Tabs>

## Force Update Wager Outcome

This endpoint allows administrators to force-update the outcome of a wager, even if it has already been decided.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                   |
| :----------- | :-------------------------------------- |
| method       | `POST`                                  |
| url          | `$baseUrl/admin/wagers/force-update-outcome` |
| Content-Type | `application/json`                      |

#### Body

| Property        | Type   | Required | Description                                    |
| --------------- | ------ | -------- | ---------------------------------------------- |
| account_id      | number | Yes      | The ID of the account                          |
| wager_reference | string | Yes      | The reference ID of the wager                  |
| outcome         | string | Yes      | The new outcome (undecided, win, half-win, loss, half-loss, push, void) |
| description     | string | No       | Description of why the outcome was changed     |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/admin/wagers/force-update-outcome" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $token" \
      -d '{
        "account_id": 2,
        "wager_reference": "wager-9",
        "outcome": "win",
        "description": "Outcome corrected after review"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const forceUpdateWagerOutcome = async (data) => {
      try {
        const response = await fetch(`${baseUrl}/admin/wagers/force-update-outcome`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          },
          body: JSON.stringify(data)
        });
        
        const result = await response.json();
        console.log('Update result:', result);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const wagerData = {
      account_id: 2,
      wager_reference: "wager-9",
      outcome: "win",
      description: "Outcome corrected after review"
    };

    forceUpdateWagerOutcome(wagerData);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def force_update_wager_outcome(token, account_id, wager_reference, outcome, description=None):
        url = f"{base_url}/admin/wagers/force-update-outcome"
        
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {token}'
        }
        
        payload = {
            'account_id': account_id,
            'wager_reference': wager_reference,
            'outcome': outcome
        }
        
        if description:
            payload['description'] = description
        
        response = requests.post(url, headers=headers, data=json.dumps(payload))

        if response.status_code == 200:
            print('Update successful:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    account_id = 2
    wager_reference = "wager-9"
    outcome = "win"
    description = "Outcome corrected after review"

    result = force_update_wager_outcome(token, account_id, wager_reference, outcome, description)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use std::error::Error;

    async fn force_update_wager_outcome(
        client: &Client, 
        base_url: &str, 
        token: &str,
        account_id: u64,
        wager_reference: &str,
        outcome: &str,
        description: Option<&str>
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/wagers/force-update-outcome", base_url);
        
        // Build the payload
        let mut payload = json!({
            "account_id": account_id,
            "wager_reference": wager_reference,
            "outcome": outcome
        });
        
        if let Some(desc) = description {
            payload["description"] = json!(desc);
        }
        
        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .json(&payload)
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Update successful: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let base_url = "your_base_url"; // Replace with actual base URL
        let token = "your_auth_token"; // Replace with actual token
        let account_id = 2;
        let wager_reference = "wager-9";
        let outcome = "win";
        let description = Some("Outcome corrected after review");

        force_update_wager_outcome(
            &client, 
            base_url, 
            token, 
            account_id, 
            wager_reference, 
            outcome, 
            description
        ).await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem value="Success">

    Http Code: `200`
    ```json
    {
      "message": "Wager outcome updated successfully"
    }
    ```

  </TabItem>
  <TabItem value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **Forbidden** <br/>
    Http Code: `403`
    ```json
    {
      "status": 403,
      "error": "Admin access required"
    }
    ```

    **Not Found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "Wager not found"
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to rollback bet",
      "data": [
        {
          "message": "User balance not sufficient to rollback bet",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_rollback": -270000,
            "current_exposure": 0,
            "exposure_after_rollback": 0
          },
          "associated_bet_ids": [
            7
          ]
        }
      ]
    }
    ```

  </TabItem>
</Tabs>