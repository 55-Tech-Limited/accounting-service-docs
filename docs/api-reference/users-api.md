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

:::note

The `reference` field is optional and will be generated if not provided.

:::

:::tip

If you have a special way of tracking your users and want to use your own reference, you can provide it in the `reference` field.

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/account/user" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "John Doe",
        "reference": "johndoe123",
        "preferences": {}
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
      preferences: {}
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
        "preferences": {}
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
            "preferences": {}
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

List users in your account with simple pagination.

### Request

| Property | Value                         |
| :------- | :---------------------------- |
| method   | `GET`                         |
| url      | `$baseUrl/account/user`       |

#### Query Parameters

| Name      | Required | Default | Description              |
| --------- | -------- | ------- | ------------------------ |
| page      | No       | 1       | Page number (1-based)    |
| per_page  | No       | 20      | Items per page (max 100) |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/account/user?page=1&per_page=2"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const listUsers = async (page = 1, perPage = 2) => {
      const res = await fetch(`${baseUrl}/account/user?page=${page}&per_page=${perPage}`);
      const data = await res.json();
      console.log(data);
    };

    listUsers();
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

    **Bad Request** <br/>
    Http Code: `400`
    ```json
    {
      "statusCode": 400,
      "message": "Invalid page"
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
