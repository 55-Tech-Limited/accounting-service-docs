---
sidebar_position: 2
description: "This guide provides information on how to create and manage users on your platform using the BetProtocol API."
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Users APIs

## Creating a User

Once you have authenticated, you can create users who will be placing bets on your platform. To create a user, you need to send a POST request to our API with the user's details. The `reference` field is optional and will be generated if not provided. The `preferences` field is also optional.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                       |
| :----------- | :-------------------------- |
| method       | `POST`                      |
| url          | `$baseUrl/account/user`     |
| Content-Type | `application/json`          |

#### Body

| Property    | Type   | Required | Default | Description                  |
| ----------- | ------ | -------- | ------- | ---------------------------- |
| name        | string | Yes      | -       | Name of the user             |
| reference   | string | No       | -       | Preferred User reference     |
| preferences | object | No       | NULL    | User preferences             |
| meta        | object | No       | NULL    | User metadata (max 10 string fields) |

:::note

The `reference` field is optional and will be generated if not provided.

:::

:::tip

If you have a special way of tracking your users and want to use your own reference, you can provide it in the `reference` field.

:::

:::note

The `meta` field allows you to store additional user metadata with the following constraints:
- Maximum of 10 fields
- All values must be strings
- Keys can only contain letters, numbers, and underscores
- Example: `{"custom_id": "12345", "source": "mobile_app", "tier": "premium"}`

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/account/user" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "John Doe",
        "reference": "johndoe123",
        "preferences": {},
        "meta": {
          "custom_id": "12345",
          "source": "web_app",
          "tier": "premium"
        }
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const createUser = async (userData) => {
      try {
        const response = await fetch(`${baseUrl}/account/user`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(userData)
        });
        const data = await response.json();
        console.log('Success:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const userData = {
      name: "John Doe",
      reference: "johndoe123",
      preferences: {},
      meta: {
        custom_id: "12345",
        source: "web_app",
        tier: "premium"
      }
    };

    createUser(userData);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def create_user(user_data):
        url = f"{base_url}/account/user"
        headers = {
            'Content-Type': 'application/json'
        }
        response = requests.post(url, headers=headers, data=json.dumps(user_data))

        if response.status_code == 201:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage:
    user_data = {
        "name": "John Doe",
        "reference": "johndoe123",
        "preferences": {},
        "meta": {
            "custom_id": "12345",
            "source": "web_app",
            "tier": "premium"
        }
    }

    create_user(user_data)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use std::error::Error;

    async fn create_user(user_data: serde_json::Value) -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let response = client.post(&format!("{}/account/user", base_url))
            .json(&user_data)
            .send()
            .await?;

        if response.status().is_success() {
            let data: serde_json::Value = response.json().await?;
            println!("Success: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let user_data = json!({
            "name": "John Doe",
            "reference": "johndoe123",
            "preferences": {},
            "meta": {
                "custom_id": "12345",
                "source": "web_app",
                "tier": "premium"
            }
        });

        create_user(user_data).await?;
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `201`
    ```json
    {
      "data": {
        "id": 2,
  "reference": "a2_user_Xhwz442NTj74z6F0",
  "created_at": "2025-09-08T12:00:00.000Z"
      },
      "message": "User created successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
      "message": [
        "name must be shorter than or equal to 256 characters",
        "name should not be empty"
      ],
      "error": "Bad Request",
      "statusCode": 400
    }
    ```

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

  </TabItem>
</Tabs>

## Get Paginated Users

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                   |
| :----------- | :---------------------- |
| method       | `GET`                   |
| url          | `$baseUrl/account/user` |
| Content-Type | `application/json`      |

#### Query Parameters

| Property   | Type   | Required | Default | Description                                           |
| ---------- | ------ | -------- | ------- | ----------------------------------------------------- |
| page       | number | No       | 1       | Page number (minimum 1)                               |
| per_page   | number | No       | 20      | Results per page (1 - 100)                            |
| search     | string | No       | -       | Case-insensitive partial match on name or reference   |
| sort_by    | string | No       | created_at | One of: id, name, reference, created_at, balance, exposure |
| sort_order | string | No       | desc    | asc or desc                                           |
| meta_*     | string | No       | -       | Filter by meta field values (e.g., meta_tier=premium, meta_source=mobile_app) |

:::note

**Meta Field Filtering**: You can filter users by their meta field values using the `meta_*` pattern:
- Use `meta_` followed by the field name and its value
- Example: `meta_tier=premium` filters users where `meta.tier` equals "premium"
- Multiple meta filters can be combined: `meta_tier=premium&meta_source=mobile_app`
- Values are matched exactly (case-sensitive)

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    # Basic pagination
    curl -X GET "$baseUrl/account/user?page=1&per_page=2"
    
    # Filter by meta fields
    curl -X GET "$baseUrl/account/user?page=1&per_page=10&meta_tier=premium&meta_source=mobile_app"
    
    # Combine with search and sorting
    curl -X GET "$baseUrl/account/user?search=john&sort_by=created_at&sort_order=asc&meta_tier=gold"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const listUsers = async (params = {}) => {
      const q = new URLSearchParams(params).toString();
      const res = await fetch(`${baseUrl}/account/user${q ? `?${q}` : ''}`);
      return res.json();
    };

    // Basic pagination
    listUsers({ page: 1, per_page: 2 }).then(console.log);
    
    // Filter by meta fields
    listUsers({
      page: 1,
      per_page: 10,
      meta_tier: 'premium',
      meta_source: 'mobile_app'
    }).then(console.log);
    
    // Combine with search and sorting
    listUsers({
      search: 'john',
      sort_by: 'created_at',
      sort_order: 'asc',
      meta_tier: 'gold'
    }).then(console.log);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def list_users(params=None):
        params = params or {}
        url = f"{base_url}/account/user"
        response = requests.get(url, params=params)
        print(response.json())

    # Basic pagination
    list_users({"page": 1, "per_page": 2})
    
    # Filter by meta fields
    list_users({
        "page": 1,
        "per_page": 10,
        "meta_tier": "premium",
        "meta_source": "mobile_app"
    })
    
    # Combine with search and sorting
    list_users({
        "search": "john",
        "sort_by": "created_at",
        "sort_order": "asc",
        "meta_tier": "gold"
    })
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use std::error::Error;

    async fn list_users(query_params: &str) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/account/user{}", base_url, query_params);
        let client = Client::new();
        let resp = client.get(&url).send().await?;
        let data: serde_json::Value = resp.json().await?;
        println!("{:?}", data);
        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        // Basic pagination
        list_users("?page=1&per_page=2").await?;
        
        // Filter by meta fields
        list_users("?page=1&per_page=10&meta_tier=premium&meta_source=mobile_app").await?;
        
        // Combine with search and sorting
        list_users("?search=john&sort_by=created_at&sort_order=asc&meta_tier=gold").await?;
        
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
      "data": {
        "items": [
          {
            "id": 1,
            "account_id": 2,
            "reference": "a2_user_4lH6u7hayvaqs6Ix",
            "name": "Jane Doe",
            "role": "user",
            "preferences": { "allow_negative_balance": true },
            "meta": { "tier": "premium", "source": "mobile_app" },
            "balance": 0,
            "exposure": 0,
            "created_at": "2025-09-08T12:00:00.000Z"
          },
          {
            "id": 2,
            "account_id": 2,
            "reference": "a2_user_Xhwz442NTj74z6F0",
            "name": "John Doe",
            "role": "user",
            "preferences": {},
            "meta": { "custom_id": "12345", "source": "web_app" },
            "balance": 0,
            "exposure": 0,
            "created_at": "2025-09-08T12:05:10.000Z"
          }
        ],
        "page": 1,
        "per_page": 2,
        "total": 12,
        "total_pages": 6
      },
      "message": "Users fetched successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
      "statusCode": 400,
      "message": [
        "sort_by must be one of the following values: id, name, reference, created_at, balance, exposure"
      ],
      "error": "Bad Request"
    }
    ```

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

  </TabItem>
</Tabs>


## Get User By Id

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                               |
| :----------- | :---------------------------------- |
| method       | `GET`                               |
| url          | `$baseUrl/account/user/:userId`     |
| Content-Type | `application/json`                  |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/account/user/:userId"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const getUserDetails = async (userId) => {
      try {
        const response = await fetch(`${baseUrl}/account/user/${userId}`, {
          method: 'GET'
        });
        if (response.ok) {
          const data = await response.json();
          console.log('Success:', data);
        } else {
          console.error('Error:', await response.text());
        }
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const userId = '12345';  // Replace with the actual user ID

    getUserDetails(userId);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_user_details(user_id):
        url = f"{base_url}/account/user/{user_id}"
        response = requests.get(url)

        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage:
    user_id = '12345'  # Replace with the actual user ID

    get_user_details(user_id)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use std::error::Error;

    async fn get_user_details(user_id: &str) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/account/user/{}", base_url, user_id);
        let client = Client::new();
        let response = client.get(&url).send().await?;

        if response.status().is_success() {
            let data: serde_json::Value = response.json().await?;
            println!("Success: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let user_id = "12345"; // Replace with the actual user ID
        get_user_details(user_id).await?;
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
      "data": {
        "id": 1,
        "account_id": 2,
        "reference": "a2_user_4lH6u7hayvaqs6Ix",
        "name": "Jane Doe",
        "role": "user",
        "preferences": {
          "allow_negative_balance": true
        },
        "meta": {
          "tier": "premium",
          "source": "mobile_app",
          "custom_id": "12345"
        },
        "balance": 0,
        "exposure": 0,
        "created_at": "2025-09-08T12:00:00.000Z"
      },
      "message": "User fetched successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "User not found"
    }
    ```

  </TabItem>
</Tabs>

## Get User By Reference

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                            |
| :----------- | :----------------------------------------------- |
| method       | `GET`                                            |
| url          | `$baseUrl/account/user/:reference/reference`     |
| Content-Type | `application/json`                               |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/account/user/:reference/reference"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const getUserDetails = async (reference) => {
      try {
        const response = await fetch(`${baseUrl}/account/user/${reference}/reference`, {
          method: 'GET'
        });
        if (response.ok) {
          const data = await response.json();
          console.log('Success:', data);
        } else {
          console.error('Error:', await response.text());
        }
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const reference = 'janedoe123';  // Replace with the actual user reference

    getUserDetails(reference);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_user_details(reference):
        url = f"{base_url}/account/user/{reference}/reference"
        response = requests.get(url)

        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage:
    reference = 'janedoe123'  # Replace with the actual user reference

    get_user_details(reference)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use std::error::Error;

    async fn get_user_details(reference: &str) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/account/user/{}/reference", base_url, reference);
        let client = Client::new();
        let response = client.get(&url).send().await?;

        if response.status().is_success() {
            let data: serde_json::Value = response.json().await?;
            println!("Success: {:?}", data);
        } else {
            println!("Error: {:?}", response.text().await?);
        }

        Ok(())
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn Error>> {
        let reference = "janedoe123"; // Replace with the actual user reference
        get_user_details(reference).await?;
        Ok(())
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
      "data": {
        "id": 1,
        "account_id": 2,
        "reference": "a2_user_4lH6u7hayvaqs6Ix",
        "name": "Jane Doe",
        "role": "user",
        "preferences": {
          "allow_negative_balance": true
        },
        "meta": {
          "tier": "premium",
          "source": "mobile_app",
          "custom_id": "12345"
        },
        "balance": 0,
        "exposure": 0,
        "created_at": "2025-09-08T12:00:00.000Z"
      },
      "message": "User fetched successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized",
      "statusCode": 401
    }
    ```

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "User not found"
    }
    ```

  </TabItem>
</Tabs>

## Update User By Id

This endpoint allows you to update a user's information including their name, preferences, and metadata.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                               |
| :----------- | :---------------------------------- |
| method       | `PATCH`                             |
| url          | `$baseUrl/account/user/:userId`     |
| Content-Type | `application/json`                  |

#### Path Parameters

| Parameter | Description           |
| --------- | --------------------- |
| userId    | The ID of the user    |

#### Body

| Property    | Type   | Required | Default | Description                  |
| ----------- | ------ | -------- | ------- | ---------------------------- |
| name        | string | No       | -       | Updated name of the user     |
| preferences | object | No       | -       | Updated user preferences     |
| meta        | object | No       | -       | Updated user metadata (max 10 string fields) |

:::note

All fields are optional in the update request. Only provided fields will be updated.

The `meta` field follows the same constraints as in user creation:
- Maximum of 10 fields
- All values must be strings  
- Keys can only contain letters, numbers, and underscores

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X PATCH "$baseUrl/account/user/123" \
      -H "Authorization: Bearer $token" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "John Smith",
        "meta": {
          "tier": "gold",
          "updated_source": "admin_panel"
        }
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const updateUser = async (userId, updateData) => {
      try {
        const response = await fetch(`${baseUrl}/account/user/${userId}`, {
          method: 'PATCH',
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(updateData)
        });
        
        const data = await response.json();
        console.log('User updated:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const userId = 123;
    const updateData = {
      name: "John Smith",
      meta: {
        tier: "gold",
        updated_source: "admin_panel"
      }
    };

    updateUser(userId, updateData);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def update_user(token, user_id, update_data):
        url = f"{base_url}/account/user/{user_id}"
        
        headers = {
            'Authorization': f'Bearer {token}',
            'Content-Type': 'application/json'
        }
        
        response = requests.patch(url, headers=headers, json=update_data)

        if response.status_code == 200:
            print('User updated successfully:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    user_id = 123
    update_data = {
        "name": "John Smith",
        "meta": {
            "tier": "gold",
            "updated_source": "admin_panel"
        }
    }

    result = update_user(token, user_id, update_data)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use std::error::Error;

    async fn update_user(
        client: &Client, 
        base_url: &str, 
        token: &str,
        user_id: u64,
        update_data: Value
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/account/user/{}", base_url, user_id);
        
        let response = client.patch(&url)
            .header("Authorization", format!("Bearer {}", token))
            .header("Content-Type", "application/json")
            .json(&update_data)
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("User updated successfully: {:?}", data);
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
        let user_id = 123;
        
        let update_data = json!({
            "name": "John Smith",
            "meta": {
                "tier": "gold",
                "updated_source": "admin_panel"
            }
        });

        update_user(&client, base_url, token, user_id, update_data).await?;
        
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
        "id": 123,
        "reference": "a2_user_Xhwz442NTj74z6F0",
        "name": "John Smith",
        "preferences": {
          "allow_negative_balance": false
        },
        "meta": {
          "tier": "gold",
          "updated_source": "admin_panel",
          "custom_id": "12345"
        },
        "updated_at": "2025-09-24T12:00:00.000Z"
      },
      "message": "User updated successfully"
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

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "User not found"
    }
    ```

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
      "message": [
        "name must be shorter than or equal to 256 characters",
        "Meta object cannot have more than 10 fields",
        "All meta values must be strings"
      ],
      "error": "Bad Request",
      "statusCode": 400
    }
    ```

  </TabItem>
</Tabs>

## Update User By Reference

This endpoint allows you to update a user's information using their reference instead of their ID.

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                          |
| :----------- | :--------------------------------------------- |
| method       | `PATCH`                                        |
| url          | `$baseUrl/account/user/:reference/reference`   |
| Content-Type | `application/json`                             |

#### Path Parameters

| Parameter | Description               |
| --------- | ------------------------- |
| reference | The reference of the user |

#### Body

| Property    | Type   | Required | Default | Description                  |
| ----------- | ------ | -------- | ------- | ---------------------------- |
| name        | string | No       | -       | Updated name of the user     |
| preferences | object | No       | -       | Updated user preferences     |
| meta        | object | No       | -       | Updated user metadata (max 10 string fields) |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X PATCH "$baseUrl/account/user/johndoe123/reference" \
      -H "Authorization: Bearer $token" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "John Smith",
        "meta": {
          "tier": "gold",
          "updated_source": "admin_panel"
        }
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const updateUserByReference = async (userReference, updateData) => {
      try {
        const response = await fetch(`${baseUrl}/account/user/${userReference}/reference`, {
          method: 'PATCH',
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(updateData)
        });
        
        const data = await response.json();
        console.log('User updated:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    // Example usage:
    const userReference = "johndoe123";
    const updateData = {
      name: "John Smith",
      meta: {
        tier: "gold",
        updated_source: "admin_panel"
      }
    };

    updateUserByReference(userReference, updateData);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def update_user_by_reference(token, user_reference, update_data):
        url = f"{base_url}/account/user/{user_reference}/reference"
        
        headers = {
            'Authorization': f'Bearer {token}',
            'Content-Type': 'application/json'
        }
        
        response = requests.patch(url, headers=headers, json=update_data)

        if response.status_code == 200:
            print('User updated successfully:', response.json())
            return response.json()
        else:
            print('Error:', response.text)
            return None

    # Example usage:
    token = 'your_auth_token'  # Replace with actual token
    user_reference = "johndoe123"
    update_data = {
        "name": "John Smith",
        "meta": {
            "tier": "gold",
            "updated_source": "admin_panel"
        }
    }

    result = update_user_by_reference(token, user_reference, update_data)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use std::error::Error;

    async fn update_user_by_reference(
        client: &Client, 
        base_url: &str, 
        token: &str,
        user_reference: &str,
        update_data: Value
    ) -> Result<(), Box<dyn Error>> {
        let url = format!("{}/account/user/{}/reference", base_url, user_reference);
        
        let response = client.patch(&url)
            .header("Authorization", format!("Bearer {}", token))
            .header("Content-Type", "application/json")
            .json(&update_data)
            .send()
            .await?;

        if response.status().is_success() {
            let data: Value = response.json().await?;
            println!("User updated successfully: {:?}", data);
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
        let user_reference = "johndoe123";
        
        let update_data = json!({
            "name": "John Smith",
            "meta": {
                "tier": "gold",
                "updated_source": "admin_panel"
            }
        });

        update_user_by_reference(&client, base_url, token, user_reference, update_data).await?;
        
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
        "id": 123,
        "reference": "johndoe123",
        "name": "John Smith",
        "preferences": {
          "allow_negative_balance": false
        },
        "meta": {
          "tier": "gold",
          "updated_source": "admin_panel",
          "custom_id": "12345"
        },
        "updated_at": "2025-09-24T12:00:00.000Z"
      },
      "message": "User updated successfully"
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

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
      "status": 404,
      "error": "User not found"
    }
    ```

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
      "message": [
        "name must be shorter than or equal to 256 characters",
        "Meta object cannot have more than 10 fields",
        "All meta values must be strings"
      ],
      "error": "Bad Request",
      "statusCode": 400
    }
    ```

  </TabItem>
</Tabs>
