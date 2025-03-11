---
sidebar_position: 2
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
| url          | `$baseUrl/api/account/user` |
| Content-Type | `application/json`          |

#### Body

| Property   | Type   | Required | Default | Description                  |
| ---------- | ------ | -------- | ------- | ---------------------------- |
| name       | string | Yes      | -       | Name of the betting business |
| reference  | string | No       | -       | Preferred User reference     |
| preference | object | No       | NULL    | Account password             |

:::note

The `reference` field is optional and will be generated if not provided.

:::

:::tip

If you have a special way of tracking your users and want to use your own reference, you can provide it in the `reference` field.

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/api/account/user" \
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
        const response = await fetch(`${baseUrl}/api/account/user`, {
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

    def create_user(base_url, user_data):
        url = f"{base_url}/api/account/user"
        headers = {
            'Content-Type': 'application/json'
        }
        response = requests.post(url, headers=headers, data=json.dumps(user_data))

        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage:
    base_url = 'https://your-api-domain.com'  # Replace with your actual base URL
    user_data = {
        "name": "John Doe",
        "reference": "johndoe123",
        "preferences": {}
    }

    create_user(base_url, user_data)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use std::error::Error;

    async fn create_user(base_url: &str, user_data: serde_json::Value) -> Result<(), Box<dyn Error>> {
        let client = Client::new();
        let response = client.post(&format!("{}/api/account/user", base_url))
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
        let base_url = "https://your-api-domain.com"; // Replace with your actual base URL
        let user_data = json!({
            "name": "John Doe",
            "reference": "johndoe123",
            "preferences": {}
        });

        create_user(base_url, user_data).await?;
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
            "id": 3,
            "reference": "johndoe123"
        },
        "message": "User created successfully"
    }
    ```

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```

    **Record already exists** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "User with reference \"janedoe123\" already exists"
    }
    ```

    **Error with body** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "name must be shorter than or equal to 256 characters",
            "name should not be empty"
        ],
        "error": "Bad Request",
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
| url          | `$baseUrl/api/account/user/:userId` |
| Content-Type | `application/json`                  |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/api/account/user/:userId"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    const getUserDetails = async (userId) => {
      try {
        const response = await fetch(`${baseUrl}/api/account/user/${userId}`, {
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
        url = f"{baseUrl}/api/account/user/{user_id}"
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
        let base_url = "https://your-api-domain.com"; // Replace with your actual base URL
        let url = format!("{}/api/account/user/{}", base_url, user_id);
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

    Http Code: `201`
    ```json
    {
        "data": {
            "id": 6,
            "account_id": 4,
            "reference": "janedoe123",
            "name": "Jane Doe",
            "role": "user",
            "preferences": null,
            "balance": 0,
            "exposure": 0
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
        "message": "Unauthorized"
    }
    ```

    **User not found** <br/>
    Http Code: `404`
    ```json
    {
        "error": "User not found"
    }
    ```

  </TabItem>
</Tabs>
