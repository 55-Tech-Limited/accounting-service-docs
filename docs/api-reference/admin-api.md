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

    **Invalid Account ID** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": "Validation failed (numeric string is expected)"
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

| Parameter        | Type   | Required | Default     | Description                                                    |
| ---------------- | ------ | -------- | ----------- | -------------------------------------------------------------- |
| page             | number | No       | 1           | Page number for pagination                                     |
| per_page         | number | No       | 20          | Number of results per page                                     |
| search           | string | No       | -           | Search term for account name/email                            |
| email_verified   | boolean| No       | -           | Filter by email verification status (true/false)             |
| sort_by          | string | No       | created_at  | Sort field (effective_odds, effective_amount, created_at)     |
| sort_direction   | string | No       | asc         | Sort direction (asc, desc)                                    |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/admin/accounts?page=1&per_page=10&search=email&email_verified=true&sort_by=created_at&sort_direction=desc" \
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
        if (options.emailVerified !== undefined) params.append('email_verified', options.emailVerified);
        if (options.sortBy) params.append('sort_by', options.sortBy);
        if (options.sortDirection) params.append('sort_direction', options.sortDirection);
        
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
      search: 'email',
      emailVerified: true,
      sortBy: 'created_at',
      sortDirection: 'desc'
    };

    getAccounts(options);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    from urllib.parse import urlencode

    def get_accounts(token, page=1, per_page=20, search=None, email_verified=None, sort_by='created_at', sort_direction='asc'):
        params = {
            'page': page,
            'per_page': per_page,
            'sort_by': sort_by,
            'sort_direction': sort_direction
        }
        
        if search:
            params['search'] = search
        
        if email_verified is not None:
            params['email_verified'] = str(email_verified).lower()
            
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

    accounts = get_accounts(token, page=1, per_page=10, search='email', email_verified=True, sort_by='created_at', sort_direction='desc')
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
        search: Option<&str>,
        email_verified: Option<bool>,
        sort_by: Option<&str>,
        sort_direction: Option<&str>
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
        
        if let Some(ev) = email_verified {
            url.push_str(&format!("email_verified={}&", ev));
        }
        
        if let Some(sb) = sort_by {
            url.push_str(&format!("sort_by={}&", sb));
        }
        
        if let Some(sd) = sort_direction {
            url.push_str(&format!("sort_direction={}&", sd));
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
            Some("email"),
            Some(true),
            Some("created_at"),
            Some("desc")
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

    **Invalid Query Parameters** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": [
        "page must be a number",
        "per_page must be a number",
        "email_verified must be a boolean value",
        "sort_by must be one of the following values: effective_odds, effective_amount, created_at",
        "sort_direction must be one of the following values: asc, desc"
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

    **Outcome Already Set** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Wager outcome already win"
    }
    ```

    **Invalid Outcome Value** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": [
        "outcome - (invalid_value) not a valid bet outcome (undecided, win, loss, push, half-win, half-loss, void)"
      ]
    }
    ```

    **Invalid Request Body** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": [
        "account_id must be a number",
        "wager_reference should not be empty"
      ]
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to set bet inactive",
      "data": [
        {
          "message": "User balance not sufficient for action",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_action": -270000,
            "current_exposure": 0,
            "exposure_after_action": 0
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

## Set Bet Inactive

This endpoint allows administrators to deactivate a bet by its ID. When a bet is set as inactive, it will trigger a rollback of the bet's financial impact, restoring user balances and exposure to their previous state.

:::warning Important Behavior

When a bet is set as inactive:
- The bet's `is_active` status is set to `false`
- A rollback transaction is performed to restore user balances
- The bet's offer status remains unchanged
- An audit trail is created with the admin user's details
- If the bet has already been settled, appropriate balance adjustments are made

:::

### Request

| Property     | Value                                    |
| :----------- | :--------------------------------------- |
| method       | `POST`                                   |
| url          | `$baseUrl/admin/bets/:betId/set-inactive` |
| Content-Type | `application/json`                       |

#### Path Parameters

| Parameter | Description              |
| --------- | ------------------------ |
| betId     | The ID of the bet to deactivate |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/admin/bets/7/set-inactive" \
      -H "Authorization: Bearer $token" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const setBetInactive = async (betId) => {
      try {
        const response = await fetch(`${baseUrl}/admin/bets/${betId}/set-inactive`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          }
        });
        
        const result = await response.json();
        console.log('Bet deactivated:', result);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    setBetInactive(7);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def set_bet_inactive(token, bet_id):
        url = f"{base_url}/admin/bets/{bet_id}/set-inactive"
        
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {token}'
        }
        
        response = requests.post(url, headers=headers)

        if response.status_code == 200:
            print('Bet deactivated successfully:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    bet_id = 7

    result = set_bet_inactive(token, bet_id)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::error::Error;

    async fn set_bet_inactive(
        client: &Client, 
        base_url: &str, 
        token: &str,
        bet_id: u64
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/bets/{}/set-inactive", base_url, bet_id);
        
        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Bet deactivated successfully: {:?}", data);
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

        set_bet_inactive(&client, base_url, token, bet_id).await?;
        
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
      "message": "Bet set inactive successfully"
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
      "error": "Bet not found"
    }
    ```

    **Already Inactive** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bet already inactive"
    }
    ```

    **Invalid Bet ID** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": "Validation failed (numeric string is expected)"
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to set bet inactive",
      "data": [
        {
          "message": "User balance not sufficient for action",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_action": -270000,
            "current_exposure": 0,
            "exposure_after_action": 0
          },
          "associated_bet_ids": [7]
        }
      ]
    }
    ```

  </TabItem>
</Tabs>

## Set Bet Active

This endpoint allows administrators to reactivate a previously deactivated bet by its ID. When a bet is set as active, it will restore the bet's financial impact on user balances and exposure.

:::warning Important Behavior

When a bet is set as active:
- The bet's `is_active` status is set to `true`
- Transaction actions are performed to restore the bet's financial impact
- The bet's offer status remains unchanged
- An audit trail is created with the admin user's details
- User balances and exposure are updated based on the bet's current state

:::

### Request

| Property     | Value                                  |
| :----------- | :------------------------------------- |
| method       | `POST`                                 |
| url          | `$baseUrl/admin/bets/:betId/set-active` |
| Content-Type | `application/json`                     |

#### Path Parameters

| Parameter | Description              |
| --------- | ------------------------ |
| betId     | The ID of the bet to reactivate |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/admin/bets/7/set-active" \
      -H "Authorization: Bearer $token" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const setBetActive = async (betId) => {
      try {
        const response = await fetch(`${baseUrl}/admin/bets/${betId}/set-active`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          }
        });
        
        const result = await response.json();
        console.log('Bet activated:', result);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    setBetActive(7);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def set_bet_active(token, bet_id):
        url = f"{base_url}/admin/bets/{bet_id}/set-active"
        
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {token}'
        }
        
        response = requests.post(url, headers=headers)

        if response.status_code == 200:
            print('Bet activated successfully:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    bet_id = 7

    result = set_bet_active(token, bet_id)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::Value;
    use std::error::Error;

    async fn set_bet_active(
        client: &Client, 
        base_url: &str, 
        token: &str,
        bet_id: u64
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/bets/{}/set-active", base_url, bet_id);
        
        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .header("Authorization", format!("Bearer {}", token))
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("Bet activated successfully: {:?}", data);
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

        set_bet_active(&client, base_url, token, bet_id).await?;
        
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
      "message": "Bet set active successfully"
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
      "error": "Bet not found"
    }
    ```

    **Already Active** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bet already active"
    }
    ```

    **Invalid Bet ID** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": "Validation failed (numeric string is expected)"
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to set bet active",
      "data": [
        {
          "message": "User balance not sufficient for action",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_action": -270000,
            "current_exposure": 0,
            "exposure_after_action": 0
          },
          "associated_bet_ids": [7]
        }
      ]
    }
    ```

  </TabItem>
</Tabs>

## Force Update Bet Override Outcome

This endpoint allows administrators to force update the override outcome of a specific bet. This is useful for correcting bet outcomes when there are disputes or errors in the original settlement.

:::warning Important Behavior

When a bet's override outcome is updated:
- The current bet's financial impact is rolled back first
- The bet's `override_outcome` is updated to the new value
- If the bet is in "requesting" or "accepting" state and the override outcome is not "undecided", the bet's offer status is changed to "expired"
- If the override outcome is set to "undecided" and the bet is "expired", the offer status is restored to "requesting" or "accepting"
- New transaction actions are performed based on the updated outcome
- An audit trail is created with the admin user's details

:::

### Request

| Property     | Value                                           |
| :----------- | :---------------------------------------------- |
| method       | `POST`                                          |
| url          | `$baseUrl/admin/bets/force-update-override-outcome` |
| Content-Type | `application/json`                              |

#### Body Parameters

| Parameter         | Type     | Required | Description                                                                                        |
| ----------------- | -------- | -------- | -------------------------------------------------------------------------------------------------- |
| account_id        | number   | Yes      | The ID of the account that owns the bet                                                           |
| bet_id            | number   | Yes      | The ID of the bet to update                                                                       |
| override_outcome  | string   | No       | The new override outcome (undecided, win, loss, push, half-win, half-loss, void, or null to remove) |
| description       | string   | No       | Optional description for the action                                                               |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/admin/bets/force-update-override-outcome" \
      -H "Authorization: Bearer $token" \
      -H "Content-Type: application/json" \
      -d '{
        "account_id": 2,
        "bet_id": 7,
        "override_outcome": "win",
        "description": "Corrected outcome after review"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const forceUpdateBetOverrideOutcome = async (data) => {
      try {
        const response = await fetch(`${baseUrl}/admin/bets/force-update-override-outcome`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          },
          body: JSON.stringify(data)
        });
        
        const result = await response.json();
        console.log('Override outcome updated:', result);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const betData = {
      account_id: 2,
      bet_id: 7,
      override_outcome: "win",
      description: "Corrected outcome after review"
    };

    forceUpdateBetOverrideOutcome(betData);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def force_update_bet_override_outcome(token, account_id, bet_id, override_outcome, description=None):
        url = f"{base_url}/admin/bets/force-update-override-outcome"
        
        headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {token}'
        }
        
        payload = {
            'account_id': account_id,
            'bet_id': bet_id,
            'override_outcome': override_outcome
        }
        
        if description:
            payload['description'] = description
        
        response = requests.post(url, headers=headers, data=json.dumps(payload))

        if response.status_code == 200:
            print('Override outcome updated successfully:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    account_id = 2
    bet_id = 7
    override_outcome = "win"
    description = "Corrected outcome after review"

    result = force_update_bet_override_outcome(token, account_id, bet_id, override_outcome, description)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use std::error::Error;

    async fn force_update_bet_override_outcome(
        client: &Client, 
        base_url: &str, 
        token: &str,
        account_id: u64,
        bet_id: u64,
        override_outcome: Option<&str>,
        description: Option<&str>
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/admin/bets/force-update-override-outcome", base_url);
        
        // Build the payload
        let mut payload = json!({
            "account_id": account_id,
            "bet_id": bet_id
        });
        
        if let Some(outcome) = override_outcome {
            payload["override_outcome"] = json!(outcome);
        }
        
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
            println!("Override outcome updated successfully: {:?}", data);
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
        let bet_id = 7;
        let override_outcome = Some("win");
        let description = Some("Corrected outcome after review");

        force_update_bet_override_outcome(
            &client, 
            base_url, 
            token, 
            account_id, 
            bet_id, 
            override_outcome, 
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
      "message": "Override outcome updated successfully"
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
      "error": "Bet not found"
    }
    ```

    **Inactive Bet** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Cannot update override outcome for inactive bet"
    }
    ```

    **Same Outcome** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Override outcome is already set to this value"
    }
    ```

    **Invalid Outcome Value** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": [
        "override_outcome - (invalid_value) not a valid bet outcome (undecided, win, loss, push, half-win, half-loss, void)"
      ]
    }
    ```

    **Invalid Request Body** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Bad Request",
      "message": [
        "account_id must be a number",
        "bet_id must be a number"
      ]
    }
    ```

    **Insufficient Balance** <br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Unable to update override outcome",
      "data": [
        {
          "message": "User balance not sufficient for action",
          "user": {
            "id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "current_balance": 0,
            "balance_after_action": -270000,
            "current_exposure": 0,
            "exposure_after_action": 0
          },
          "associated_bet_ids": [7]
        }
      ]
    }
    ```

  </TabItem>
</Tabs>