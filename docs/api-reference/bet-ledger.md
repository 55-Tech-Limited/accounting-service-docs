---
sidebar_position: 4
description: "Efficiently manage and interact with your betting system using the Bet Ledger API."
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Bet-Ledger APIs

## Make Bet Offer

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                          |
| :----------- | :----------------------------- |
| method       | `POST`                         |
| url          | `$baseUrl/bets/make-offer`     |
| Content-Type | `application/json`             |

#### Body

| Property                       | Type         | Required | Default | Description               |
| ------------------------------ | ------------ | -------- | ------- | ------------------------- |
| wager_reference                | string       | Yes      | -       | Wager reference           |
| requesting_odds                | number (>=1) | Yes      | -       | Requesting odds           |
| requesting_amount              | number (>0)  | Yes      | -       | Requesting amount         |
| requesting_user_id             | number       | No       | -       | Requesting user id        |
| requesting_user_reference      | string       | No       | -       | Requesting user reference |
| allowed_acceptor_ids           | number[]     | No       | -       | Array of user IDs allowed to accept this bet |
| allowed_acceptor_references    | string[]     | No       | -       | Array of user references allowed to accept this bet |
| meta                           | object       | No       | -       | User specified meta data  |

:::info

Either the `requesting_user_id` or `requesting_user_reference` must be provided

:::

:::note Allowed Acceptors

You can restrict who can accept your bet offer by providing either `allowed_acceptor_ids` or `allowed_acceptor_references`. If both are provided, users matching either condition can accept the bet.

If no allowed acceptor fields are provided, the bet offer will be open to all users.

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    # Open bet offer (anyone can accept)
    curl -X POST "$baseUrl/bets/make-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "wager_reference": "example_wager_reference",
        "requesting_odds": 2,
        "requesting_amount": 100,
        "requesting_user_id": 12345
      }'
    
    # Restricted bet offer (only specific users can accept)
    curl -X POST "$baseUrl/bets/make-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "wager_reference": "example_wager_reference",
        "requesting_odds": 2,
        "requesting_amount": 100,
        "requesting_user_id": 12345,
        "allowed_acceptor_ids": [67890, 11111],
        "allowed_acceptor_references": ["user_abc", "user_xyz"]
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    // Open bet offer (anyone can accept)
    function makeOpenBetOffer() {
      const url = `${baseUrl}/bets/make-offer`;
      
      const data = {
        wager_reference: "example_wager_reference",
        requesting_odds: 2,
        requesting_amount: 100,
        requesting_user_id: 12345
      };

      fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Restricted bet offer (only specific users can accept)
    function makeRestrictedBetOffer() {
      const url = `${baseUrl}/bets/make-offer`;
      
      const data = {
        wager_reference: "example_wager_reference",
        requesting_odds: 2,
        requesting_amount: 100,
        requesting_user_id: 12345,
        allowed_acceptor_ids: [67890, 11111],
        allowed_acceptor_references: ["user_abc", "user_xyz"]
      };

      fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    makeOpenBetOffer();      // Open to all users
    makeRestrictedBetOffer(); // Restricted to specific users
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def make_open_bet_offer():
        """Make a bet offer open to all users"""
        url = f"{baseUrl}/bets/make-offer"
        data = {
            "wager_reference": "example_wager_reference",
            "requesting_odds": 2,
            "requesting_amount": 100,
            "requesting_user_id": 12345
        }
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    def make_restricted_bet_offer():
        """Make a bet offer restricted to specific users"""
        url = f"{baseUrl}/bets/make-offer"
        data = {
            "wager_reference": "example_wager_reference",
            "requesting_odds": 2,
            "requesting_amount": 100,
            "requesting_user_id": 12345,
            "allowed_acceptor_ids": [67890, 11111],
            "allowed_acceptor_references": ["user_abc", "user_xyz"]
        }
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    make_open_bet_offer()       # Open to all users
    make_restricted_bet_offer() # Restricted to specific users
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn make_open_bet_offer() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/make-offer", base_url);
        let client = Client::new();
        let data = json!({
            "wager_reference": "example_wager_reference",
            "requesting_odds": 2,
            "requesting_amount": 100,
            "requesting_user_id": 12345
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    #[tokio::main]
    async fn make_restricted_bet_offer() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/make-offer", base_url);
        let client = Client::new();
        let data = json!({
            "wager_reference": "example_wager_reference",
            "requesting_odds": 2,
            "requesting_amount": 100,
            "requesting_user_id": 12345,
            "allowed_acceptor_ids": [67890, 11111],
            "allowed_acceptor_references": ["user_abc", "user_xyz"]
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    // make_open_bet_offer();       // Open to all users
    // make_restricted_bet_offer(); // Restricted to specific users
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
            "bet_id": 2,
            "wager_reference": "wager-4",
            "wager_id": 2
        },
        "message": "Bet offer placed successfully"
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
        "status": 404,
        "error": "Requesting user not found"
    }
    ```

    **Account not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Account not found"
    }
    ```

    **Missing user identifier** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "requesting_user_id or requesting_user_reference must be defined"
    }
    ```

    **Wager already decided** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Wager outcome already decided"
    }
    ```

    **Insufficient balance** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Effective balance too low",
        "data": {
            "id": 1,
            "balance": 101900,
            "exposure": 60300,
            "effective_balance": 41600,
            "preferences": {
                "allow_negative_balance": false
            },
            "account_preferences": {
                "allow_negative_balance": false
            }
        }
    }
    ```

    :::tip Understanding Exposure

    For detailed information about how exposure and effective balance are calculated, see our [Exposure Calculations Guide](/docs/guides/exposure-calculations).

    :::

    **Error with body** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "requesting_user_id must not be less than 1",
            "requesting_user_id must be a number conforming to the specified constraints",
            "requesting_user_reference should not be empty",
            "requesting_user_reference must be a string",
            "requesting_odds must not be less than 1",
            "requesting_odds must be a number conforming to the specified constraints",
            "requesting_amount must not be less than 1",
            "requesting_amount must be a number conforming to the specified constraints",
            "wager_reference must be longer than or equal to 1 characters"
        ],
        "error": "Bad Request"
    }
    ```

  </TabItem>
</Tabs>

## Accept Bet Offer

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                            |
| :----------- | :------------------------------- |
| method       | `POST`                           |
| url          | `$baseUrl/bets/accept-offer`     |
| Content-Type | `application/json`               |

#### Body

| Property                  | Type         | Required | Default | Description               |
| ------------------------- | ------------ | -------- | ------- | ------------------------- |
| wager_reference           | string       | No       | -       | Wager reference           |
| bet_id                    | number       | No       | -       | Bet ID (alternative to wager_reference) |
| maximum_odds              | number (>=1) | Yes      | -       | Requesting odds           |
| accepting_amount          | number (>0)  | Yes      | -       | Requesting amount         |
| accepting_user_id         | number       | No       | -       | Accepting user id         |
| accepting_user_reference  | number       | No       | -       | Accepting user reference  |
| requesting_user_id        | number       | No       | -       | Requesting user id        |
| requesting_user_reference | number       | No       | -       | Requesting user reference |
| meta                      | json         | No       | -       | User specified meta data  |

:::info

Either the `accepting_user_id` or `accepting_user_reference` must be provided to specify requesting user.

:::

:::info

Either the `requesting_user_id` or `requesting_user_reference` can be provided to specify requesting user. If both are omitted, the system would accept bets matching any user.

:::

:::note

Either `wager_reference` or `bet_id` must be provided. If `bet_id` is specified, the `wager_reference` field will be ignored.

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    # Using wager_reference
    curl -X POST "$baseUrl/bets/accept-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "wager_reference": "example_wager_reference",
        "maximum_odds": 2,
        "accepting_amount": 100,
        "accepting_user_id": 12345,
        "meta": {}
      }'
    
    # Using bet_id (wager_reference will be ignored)
    curl -X POST "$baseUrl/bets/accept-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "bet_id": 456,
        "maximum_odds": 2,
        "accepting_amount": 100,
        "accepting_user_id": 12345,
        "meta": {}
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function acceptBetOffer(useBetId = false) {
      const url = `${baseUrl}/bets/accept-offer`;
      
      const data = {
        maximum_odds: 2,
        accepting_amount: 100,
        accepting_user_id: 12345,
        meta: {}
      };
      
      // Add either wager_reference or bet_id
      if (useBetId) {
        data.bet_id = 456;
      } else {
        data.wager_reference = "example_wager_reference";
      }

      fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    acceptBetOffer(false); // Using wager_reference
    acceptBetOffer(true);  // Using bet_id
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def accept_bet_offer():
        url = f"{baseUrl}/bets/accept-offer"
        data = {
            "wager_reference": "example_wager_reference",
            "maximum_odds": 2,
            "accepting_amount": 100,
            "accepting_user_id": 12345,
            "meta": {}
        }
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    accept_bet_offer()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn accept_bet_offer() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/accept-offer", base_url);
        let client = Client::new();
        let data = json!({
            "wager_reference": "example_wager_reference",
            "maximum_odds": 2,
            "accepting_amount": 100,
            "accepting_user_id": 12345,
            "meta": {}
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    accept_bet_offer();
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `201`
    ```json
    {
        "data": [
            {
                "bet_id": 2,
                "requesting_user_reference": "a4_user_2YRDqo69dO4DGdin",
                "requesting_user_id": 4,
                "accepted_amount": 300,
                "accepted_odds": 3,
                "wager_reference": "wager-4",
                "wager_id": 2
            }
        ],
        "message": "Bet offer accepted successfully"
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
        "status": 404,
        "error": "Accepting user not found"
    }
    ```

    **Account not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Account not found"
    }
    ```

    **Wager not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Wager with reference not found"
    }
    ```

    **Missing user identifier** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "accepting_user_id or accepting_user_reference must be defined"
    }
    ```

    **Requesting user does not exist** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Requesting user does not exist"
    }
    ```

    **Too much accepted amount** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Accepting amount greater than the maximum amount of \"300\""
    }
    ```

    **Insufficient balance** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Effective balance too low",
        "data": {
            "id": 2,
            "balance": 0,
            "exposure": 0,
            "effective_balance": 0,
            "preferences": {},
            "account_preferences": {
                "allow_negative_balance": false
            }
        }
    }
    ```

    **Error with body** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "accepting_user_id must not be less than 1",
            "accepting_user_id must be a number conforming to the specified constraints",
            "accepting_user_reference should not be empty",
            "accepting_user_reference must be a string",
            "accepting_amount must not be less than 1",
            "accepting_amount must be a number conforming to the specified constraints",
            "maximum_odds must not be less than 1",
            "maximum_odds must be a number conforming to the specified constraints",
            "wager_reference must be longer than or equal to 1 characters"
        ],
        "error": "Bad Request",
    }
    ```

  </TabItem>
</Tabs>

## Open Bets

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                    |
| :----------- | :----------------------- |
| method       | `GET`                    |
| url          | `$baseUrl/bets/open-bets`     |
| Content-Type | `application/json`       |

#### Query Params

| Property                   | Type                                                     | Required | Default      | Description                                |
| -------------------------- | -------------------------------------------------------- | -------- | ------------ | ------------------------------------------ |
| page                       | number                                                   | No       | 1            | Current page                               |
| per_page                   | number                                                   | No       | 20           | Number of items per page                   |
| wager_references           | string                                                   | No       | -            | Comma separated wager references           |
| account_ids                | number(comma separated)                                  | No       | -            | Comma separated account ids                |
| requesting_user_ids        | number(comma separated)                                  | No       | -            | Comma separated requesting user ids        |
| requesting_user_references | string(comma separated)                                  | No       | -            | Comma separated requesting user references |
| created_after              | number(timestamp) \| DateString                          | No       | -            | Filter result from this date               |
| created_before             | number(timestamp) \| DateString                          | No       | -            | Filter result to this date                 |
| minimum_odds               | number                                                   | No       | 1            | Minimum odds                               |
| maximum_odds               | number                                                   | No       | -            | Maximum odds                               |
| minimum_amount             | number                                                   | No       | -            | Minimum amount                             |
| maximum_amount             | number                                                   | No       | -            | Maximum amount                             |
| sort_by                    | "requesting_odds" \| "requesting_amount" \| "created_at" | No       | "created_at" | Sort by                                    |
| sort_direction             | "asc" \| "desc"                                          | No       | "asc"        | Sort direction                             |

:::info

`user_ids` and `user_references` can be used in combination. The results would be merged

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/bets/open-bets" \
      -H "Content-Type: application/json" \
      -G \
      --data-urlencode "page=1" \
      --data-urlencode "per_page=20" \
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function getOpenBets() {
      const url = new URL(`${baseUrl}/bets/open-bets`);
      const params = {
        page: 1,
        per_page: 20,
        sort_by: "created_at",
        sort_direction: "asc"
      };

      Object.keys(params).forEach(key => url.searchParams.append(key, params[key]));

      fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    getOpenBets();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_open_bets():
        base_url = "your_base_url_here"
        url = f"{base_url}/bets/open-bets"
        params = {
            "page": 1,
            "per_page": 20,
            "sort_by": "created_at",
            "sort_direction": "asc"
        }

        response = requests.get(url, headers={"Content-Type": "application/json"}, params=params)
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    get_open_bets()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use tokio;

    #[tokio::main]
    async fn get_open_bets() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/open-bets", base_url);
        let client = Client::new();
        let params = [
            ("page", "1"),
            ("per_page", "20"),
            ("sort_by", "created_at"),
            ("sort_direction", "asc")
        ];

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .query(&params)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    get_open_bets();
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "data": [
            {
                "id": 3,
                "account_id": 4,
                "offer_status": "requesting",
                "wager_id": 2,
                "wager_reference": "wager-4",
                "requesting_user_id": 4,
                "requesting_user_reference": "a4_user_2YRDqo69dO4DGdin",
                "requesting_odds": 3,
                "requesting_amount": 300,
                "allowed_acceptor_ids": [67890, 11111],
                "created_at": "2025-03-11T16:11:08.156Z"
            }
        ],
        "per_page": 20,
        "page": 1,
        "total": 1,
        "from": 1,
        "to": 1,
        "last_page": 1,
        "total_requesting_amount": 300
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


    **Error with query params** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "sort_by must be one of the following values: requesting_odds, requesting_amount, created_at",
            "sort_direction must be one of the following values: asc, desc"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
</Tabs>

## Bet History

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                       |
| :----------- | :-------------------------- |
| method       | `GET`                       |
| url          | `$baseUrl/bets/history`     |
| Content-Type | `application/json`          |

#### Query Params

| Property                       | Type                                                                            | Required | Default      | Description                      |
| ------------------------------ | ------------------------------------------------------------------------------- | -------- | ------------ | -------------------------------- |
| page                           | number                                                                          | No       | 1            | Current page                     |
| per_page                       | number                                                                          | No       | 20           | Number of items per page         |
| wager_references               | string                                                                          | No       | -            | Comma separated wager references |
| account_ids                    | number(comma separated)                                                         | No       | -            | Comma separated account ids      |
| user_ids                       | number(comma separated)                                                         | No       | -            | Comma separated user ids         |
| user_references                | string(comma separated)                                                         | No       | -            | Comma separated user references  |
| bet_outcomes                   | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void" | No       | -            | Comma separated bet outcomes     |
| created_after                  | number(timestamp) \| DateString                                                 | No       | -            | Filter result from this date     |
| created_before                 | number(timestamp) \| DateString                                                 | No       | -            | Filter result to this date       |
| include_bet_trails             | boolean                                                                         | No       | false        | Include bet trails in response   |
| include_bet_trails_transactions| boolean                                                                         | No       | false        | Include bet trail transactions   |
| sort_by                        | "effective_odds" \| "effective_amount" \| "created_at"                          | No       | "created_at" | Sort by                          |
| sort_direction                 | "asc" \| "desc"                                                                 | No       | "asc"        | Sort direction                   |

:::info

`user_ids` and `user_references` can be used in combination. The results would be merged

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/bets/history" \
      -H "Content-Type: application/json" \
      -G \
      --data-urlencode "page=1" \
      --data-urlencode "per_page=20" \
      --data-urlencode "sort_by=created_at" \
      --data-urlencode "sort_direction=asc"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function getBetHistory() {
      const url = new URL(`${baseUrl}/bets/history`);
      const params = {
        page: 1,
        per_page: 20,
        sort_by: "created_at",
        sort_direction: "asc"
      };

      Object.keys(params).forEach(key => url.searchParams.append(key, params[key]));

      fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    getBetHistory();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_bet_history():
        base_url = "your_base_url_here"
        url = f"{base_url}/bets/history"
        params = {
            "page": 1,
            "per_page": 20,
            "sort_by": "created_at",
            "sort_direction": "asc"
        }

        response = requests.get(url, headers={"Content-Type": "application/json"}, params=params)
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    get_bet_history()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use tokio;

    #[tokio::main]
    async fn get_bet_history() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/history", base_url);
        let client = Client::new();
        let params = [
            ("page", "1"),
            ("per_page", "20"),
            ("sort_by", "created_at"),
            ("sort_direction", "asc")
        ];

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .query(&params)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    get_bet_history();
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "data": [
            {
                "id": 2,
                "requesting_user_id": 4,
                "requesting_user_reference": "a4_user_2YRDqo69dO4DGdin",
                "accepting_user_id": 4,
                "accepting_user_reference": "a4_user_2YRDqo69dO4DGdin",
                "offer_status": "accepted",
                "effective_amount": 300,
                "effective_odds": 3,
                "override_outcome": null,
                "is_active": true,
                "created_at": "2025-03-11T15:50:33.601Z",
                "wager": {
                    "id": 2,
                    "reference": "wager-4",
                    "outcome": "undecided",
                    "account_id": 4,
                    "created_at": "2025-03-11T15:50:33.601Z",
                    "updated_at": "2025-03-11T15:50:33.601Z"
                },
                "bet_trails": [
                    {
                        "id": 4,
                        "bet_id": 2,
                        "account_id": 4,
                        "wager_id": 2,
                        "offer_status": "requesting",
                        "description": "Requesting for bet",
                        "outcome": "undecided",
                        "override_outcome": null,
                        "requesting_odds": 3,
                        "requesting_amount": 300,
                        "accepting_odds": null,
                        "accepting_amount": null,
                        "effective_odds": null,
                        "effective_amount": null,
                        "created_at": "2025-03-11T15:50:33.601Z",
                        "transactions": []
                    },
                    {
                        "id": 6,
                        "bet_id": 2,
                        "account_id": 4,
                        "wager_id": 2,
                        "offer_status": "accepted",
                        "description": "Bet offer accepted",
                        "outcome": "undecided",
                        "override_outcome": null,
                        "requesting_odds": 3,
                        "requesting_amount": 300,
                        "accepting_odds": 3,
                        "accepting_amount": 300,
                        "effective_odds": null,
                        "effective_amount": null,
                        "created_at": "2025-03-11T16:20:43.304Z",
                        "transactions": []
                    }
                ]
            },
            {
                "id": 3,
                "requesting_user_id": 4,
                "requesting_user_reference": "a4_user_2YRDqo69dO4DGdin",
                "accepting_user_id": null,
                "accepting_user_reference": null,
                "offer_status": "requesting",
                "effective_amount": null,
                "effective_odds": null,
                "override_outcome": null,
                "is_active": true,
                "created_at": "2025-03-11T16:11:08.156Z",
                "wager": {
                      "id": 2,
                      "reference": "wager-4",
                      "outcome": "undecided",
                      "account_id": 4,
                      "created_at": "2025-03-11T15:50:33.601Z",
                      "updated_at": "2025-03-11T15:50:33.601Z"
                },
                "bet_trails": [
                    {
                        "id": 5,
                        "bet_id": 3,
                        "account_id": 4,
                        "wager_id": 2,
                        "offer_status": "requesting",
                        "description": "Requesting for bet",
                        "outcome": "undecided",
                        "override_outcome": null,
                        "requesting_odds": 3,
                        "requesting_amount": 300,
                        "accepting_odds": null,
                        "accepting_amount": null,
                        "effective_odds": null,
                        "effective_amount": null,
                        "created_at": "2025-03-11T16:11:08.156Z",
                        "transactions": []
                    }
                ]
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

:::info Field Description

**is_active**: A boolean field indicating whether the bet is currently active or inactive. Active bets (true) can be accepted by other users and appear in open bet listings. Inactive bets (false) cannot be accepted and are excluded from open bet results.

:::

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```


    **Error with query params** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "sort_by must be one of the following values: effective_odds, effective_amount, created_at",
            "($value) not a valid bet outcome (undecided, win, loss, push, half-win, half-loss)"
        ],
        "error": "Bad Request"
    }
    ```

  </TabItem>
</Tabs>

## Update wager outcome

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                    |
| :----------- | :--------------------------------------- |
| method       | `POST`                                   |
| url          | `$baseUrl/bets/update-wager-outcome`     |
| Content-Type | `application/json`                       |

#### Body

| Property  | Type                                                                            | Required | Default | Description     |
| --------- | ------------------------------------------------------------------------------- | -------- | ------- | --------------- |
| reference | string                                                                          | Yes      | -       | Wager reference |
| outcome   | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void" | Yes      | -       | Bet outcomes    |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/bets/update-wager-outcome" \
      -H "Content-Type: application/json" \
      -d '{
        "reference": "example_wager_reference",
        "outcome": "win"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function updateWagerOutcome() {
      const url = `${baseUrl}/bets/update-wager-outcome`;
      const data = {
        reference: "example_wager_reference",
        outcome: "win"
      };

      fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    updateWagerOutcome();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def update_wager_outcome():
        base_url = "your_base_url_here"
        url = f"{base_url}/bets/update-wager-outcome"
        data = {
            "reference": "example_wager_reference",
            "outcome": "win"
        }
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    update_wager_outcome()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn update_wager_outcome() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/update-wager-outcome", base_url);
        let client = Client::new();
        let data = json!({
            "reference": "example_wager_reference",
            "outcome": "win"
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    update_wager_outcome();
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `201`
    ```json
    {
        "message": "Wager outcome updated successfully"
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

    **Wager not found** <br/>
    Http Code: `404`
    ```json
    {
        "error": "Wager with reference does not exist"
    }
    ```

    **Wager not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Wager with reference does not exist"
    }
    ```

    **Wager already decided** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Wager has already been decided"
    }
    ```

    **Error with body** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "reference should not be empty",
            "outcome should not be empty",
            "($value) not a valid bet outcome (undecided, win, loss, push, half-win, half-loss, void)"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
</Tabs>

## Cancel Bet Offer

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                            |
| :----------- | :------------------------------- |
| method       | `POST`                           |
| url          | `$baseUrl/bets/cancel-offer`     |
| Content-Type | `application/json`               |

#### Body

| Property                  | Type      | Required | Default | Description                                       |
| ------------------------- | --------- | -------- | ------- | ------------------------------------------------- |
| requesting_user_id        | number    | No       | -       | Requesting user id                                |
| requesting_user_reference | string    | No       | -       | Requesting user reference                         |
| wager_references          | string[]  | No       | []      | Array of wager references to target              |
| bet_id                    | number    | No       | -       | Bet ID (alternative to wager_references)         |
| cancel_amount             | number    | No       | -       | Specific amount to cancel (partial cancel)       |
| cancel_all                | boolean   | No       | false   | Cancel all matching bets                          |
| minimum_odds              | number    | No       | -       | Only cancel bets with odds &gt;= this value         |
| maximum_odds              | number    | No       | -       | Only cancel bets with odds &lt;= this value         |

:::info

Either the `requesting_user_id` or `requesting_user_reference` must be provided to identify the requesting user.

:::

:::info

Either `cancel_amount` must be provided OR `cancel_all` must be set to `true`. If both are provided, `cancel_all` takes precedence.

:::

:::note

Either `wager_references` array or `bet_id` must be provided. If `bet_id` is specified, odds filtering (`minimum_odds`, `maximum_odds`) is ignored and the specific bet is targeted regardless of its odds.

:::

:::warning Important Notes

- Only open (requesting status) bets can be canceled
- Accepted bets cannot be canceled
- Partial cancellation creates a new bet for the remaining amount
- User exposure is automatically reduced by the canceled amount
- All cancellations are recorded in the bet trail for audit purposes
- **Odds filtering**: Use `minimum_odds` and/or `maximum_odds` to target specific bet ranges
- **Wager references**: Multiple wager references can be processed in a single request
- **Bet ID priority**: When `bet_id` is provided, odds filters are ignored

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    # Cancel specific amount using wager references
    curl -X POST "$baseUrl/bets/cancel-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "requesting_user_reference": "user123",
        "wager_references": ["wager456", "wager789"],
        "cancel_amount": 150
      }'
    
    # Cancel specific amount using bet ID (ignores odds filters)
    curl -X POST "$baseUrl/bets/cancel-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "requesting_user_id": 123,
        "bet_id": 789,
        "cancel_amount": 50
      }'
    
    # Cancel all bets with odds filtering
    curl -X POST "$baseUrl/bets/cancel-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "requesting_user_reference": "user123",
        "wager_references": ["wager456"],
        "cancel_all": true,
        "minimum_odds": 2.0,
        "maximum_odds": 4.0
      }'
    
    # Cancel bets above minimum odds only
    curl -X POST "$baseUrl/bets/cancel-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "requesting_user_reference": "user123",
        "wager_references": ["wager456", "wager789"],
        "cancel_all": true,
        "minimum_odds": 2.5
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function cancelBetOffer(options = {}) {
      const {
        useUserId = false,
        userId = 123,
        userReference = "user123",
        wagerReferences = ["wager456"],
        betId = null,
        cancelAmount = null,
        cancelAll = false,
        minimumOdds = null,
        maximumOdds = null
      } = options;

      const url = `${baseUrl}/bets/cancel-offer`;
      
      // Build request data
      const data = {};

      // Add user identifier
      if (useUserId) {
        data.requesting_user_id = userId;
      } else {
        data.requesting_user_reference = userReference;
      }

      // Add target identification
      if (betId) {
        data.bet_id = betId;
        // When bet_id is used, odds filters are ignored
      } else {
        data.wager_references = wagerReferences;
        
        // Add odds filters if provided
        if (minimumOdds !== null) {
          data.minimum_odds = minimumOdds;
        }
        if (maximumOdds !== null) {
          data.maximum_odds = maximumOdds;
        }
      }

      // Add cancellation type
      if (cancelAmount !== null) {
        data.cancel_amount = cancelAmount;
      } else {
        data.cancel_all = cancelAll;
      }

      return fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        return data;
      })
      .catch(error => {
        console.error('Error:', error);
        throw error;
      });
    }

    // Example usage
    
    // Partial cancellation with multiple wagers
    cancelBetOffer({
      userReference: "user123",
      wagerReferences: ["wager456", "wager789"],
      cancelAmount: 100
    });

    // Cancel all bets with odds filtering
    cancelBetOffer({
      userReference: "user123",
      wagerReferences: ["wager456"],
      cancelAll: true,
      minimumOdds: 2.0,
      maximumOdds: 5.0
    });

    // Cancel specific bet by ID (ignores odds filters)
    cancelBetOffer({
      useUserId: true,
      userId: 123,
      betId: 789,
      cancelAmount: 50
    });

    // Cancel all high odds bets only
    cancelBetOffer({
      userReference: "user123",
      wagerReferences: ["wager456"],
      cancelAll: true,
      minimumOdds: 3.0
    });
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json
    from typing import Optional, List

    def cancel_bet_offer(
        use_user_id: bool = False,
        user_id: int = 123,
        user_reference: str = "user123",
        wager_references: List[str] = None,
        bet_id: Optional[int] = None,
        cancel_amount: Optional[float] = None,
        cancel_all: bool = False,
        minimum_odds: Optional[float] = None,
        maximum_odds: Optional[float] = None
    ):
        url = f"{baseUrl}/bets/cancel-offer"
        
        # Build request data
        data = {}
        
        # Add user identifier
        if use_user_id:
            data["requesting_user_id"] = user_id
        else:
            data["requesting_user_reference"] = user_reference
        
        # Add target identification
        if bet_id:
            data["bet_id"] = bet_id
            # When bet_id is used, odds filters are ignored
        else:
            data["wager_references"] = wager_references or ["wager456"]
            
            # Add odds filters if provided
            if minimum_odds is not None:
                data["minimum_odds"] = minimum_odds
            if maximum_odds is not None:
                data["maximum_odds"] = maximum_odds
        
        # Add cancellation type
        if cancel_amount is not None:
            data["cancel_amount"] = cancel_amount
        else:
            data["cancel_all"] = cancel_all
        
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 201:
            print('Success:', response.json())
        else:
            print('Error:', response.text)
        
        return response

    # Example usage
    
    # Partial cancellation with multiple wagers
    cancel_bet_offer(
        user_reference="user123",
        wager_references=["wager456", "wager789"],
        cancel_amount=100
    )
    
    # Cancel all bets with odds filtering
    cancel_bet_offer(
        user_reference="user123",
        wager_references=["wager456"],
        cancel_all=True,
        minimum_odds=2.0,
        maximum_odds=5.0
    )
    
    # Cancel specific bet by ID
    cancel_bet_offer(
        use_user_id=True,
        user_id=123,
        bet_id=789,
        cancel_amount=50
    )
    
    # Cancel only high odds bets
    cancel_bet_offer(
        user_reference="user123",
        wager_references=["wager456"],
        cancel_all=True,
        minimum_odds=3.0
    )
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::{json, Value};
    use tokio;

    struct CancelBetOfferOptions {
        use_user_id: bool,
        user_id: Option<u32>,
        user_reference: Option<String>,
        wager_references: Option<Vec<String>>,
        bet_id: Option<u32>,
        cancel_amount: Option<f64>,
        cancel_all: bool,
        minimum_odds: Option<f64>,
        maximum_odds: Option<f64>,
    }

    #[tokio::main]
    async fn cancel_bet_offer(options: CancelBetOfferOptions) {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/cancel-offer", base_url);
        let client = Client::new();
        
        let mut data = json!({});
        
        // Add user identifier
        if options.use_user_id {
            data["requesting_user_id"] = json!(options.user_id.unwrap_or(123));
        } else {
            data["requesting_user_reference"] = json!(options.user_reference.unwrap_or("user123".to_string()));
        }
        
        // Add target identification
        if let Some(bet_id) = options.bet_id {
            data["bet_id"] = json!(bet_id);
            // When bet_id is used, odds filters are ignored
        } else {
            data["wager_references"] = json!(options.wager_references.unwrap_or(vec!["wager456".to_string()]));
            
            // Add odds filters if provided
            if let Some(min_odds) = options.minimum_odds {
                data["minimum_odds"] = json!(min_odds);
            }
            if let Some(max_odds) = options.maximum_odds {
                data["maximum_odds"] = json!(max_odds);
            }
        }
        
        // Add cancellation type
        if let Some(cancel_amount) = options.cancel_amount {
            data["cancel_amount"] = json!(cancel_amount);
        } else {
            data["cancel_all"] = json!(options.cancel_all);
        }

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    
    // Partial cancellation with multiple wagers
    cancel_bet_offer(CancelBetOfferOptions {
        use_user_id: false,
        user_id: None,
        user_reference: Some("user123".to_string()),
        wager_references: Some(vec!["wager456".to_string(), "wager789".to_string()]),
        bet_id: None,
        cancel_amount: Some(100.0),
        cancel_all: false,
        minimum_odds: None,
        maximum_odds: None,
    });
    
    // Cancel all with odds filtering
    cancel_bet_offer(CancelBetOfferOptions {
        use_user_id: false,
        user_id: None,
        user_reference: Some("user123".to_string()),
        wager_references: Some(vec!["wager456".to_string()]),
        bet_id: None,
        cancel_amount: None,
        cancel_all: true,
        minimum_odds: Some(2.0),
        maximum_odds: Some(5.0),
    });
    
    // Cancel specific bet by ID
    cancel_bet_offer(CancelBetOfferOptions {
        use_user_id: true,
        user_id: Some(123),
        user_reference: None,
        wager_references: None,
        bet_id: Some(789),
        cancel_amount: Some(50.0),
        cancel_all: false,
        minimum_odds: None,
        maximum_odds: None,
    });
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    **Single bet cancellation**<br/>
    Http Code: `201`
    ```json
    {
        "message": "Bet offer canceled successfully",
        "data": [
            {
                "bet_id": 123,
                "wager_reference": "wager456",
                "wager_id": 2
            }
        ]
    }
    ```

    **Multiple bets cancellation with odds filtering**<br/>
    Http Code: `201`
    ```json
    {
        "message": "Bet offer canceled successfully",
        "data": [
            {
                "bet_id": 123,
                "wager_reference": "wager456",
                "wager_id": 2
            },
            {
                "bet_id": 124,
                "wager_reference": "wager456",
                "wager_id": 2
            },
            {
                "bet_id": 125,
                "wager_reference": "wager789",
                "wager_id": 3
            }
        ]
    }
    ```

    **Partial cancellation (creates new bet for remaining amount)**<br/>
    Http Code: `201`
    ```json
    {
        "message": "Bet offer canceled successfully",
        "data": [
            {
                "bet_id": 126,
                "wager_reference": "wager456",
                "wager_id": 2,
                "note": "Partial cancellation - remaining amount: 50"
            }
        ]
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
        "status": 404,
        "error": "Requesting user not found"
    }
    ```

    **Wager not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Wager with reference not found"
    }
    ```

    **No cancelable bets** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "No cancelable bets found"
    }
    ```

    **Missing parameters** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "requesting_user_id or requesting_user_reference must be defined"
    }
    ```

    **Missing cancellation parameters** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Either cancel_all must be true or cancel_amount must be provided"
    }
    ```

    **Cancel amount too high** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Cancel amount greater than the maximum cancelable amount of \"100\""
    }
    ```

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "requesting_user_reference should not be empty",
            "wager_reference should not be empty",
            "cancel_amount must be a positive number"
        ],
        "error": "Bad Request"
    }
    ```

  </TabItem>
</Tabs>

## Override Bet Outcome

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                                    |
| :----------- | :--------------------------------------- |
| method       | `POST`                                   |
| url          | `$baseUrl/bets/override-outcome`         |
| Content-Type | `application/json`                       |

#### Body

| Property          | Type                                                                            | Required | Default | Description               |
| ----------------- | ------------------------------------------------------------------------------- | -------- | ------- | ------------------------- |
| bet_id            | number                                                                          | Yes      | -       | Bet ID to override        |
| override_outcome  | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void" | Yes      | -       | Override bet outcome      |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/bets/override-outcome" \
      -H "Content-Type: application/json" \
      -d '{
        "bet_id": 123,
        "override_outcome": "win"
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function overrideBetOutcome() {
      const url = `${baseUrl}/bets/override-outcome`;
      const data = {
        bet_id: 123,
        override_outcome: "win"
      };

      fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    overrideBetOutcome();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def override_bet_outcome():
        base_url = "your_base_url_here"
        url = f"{base_url}/bets/override-outcome"
        data = {
            "bet_id": 123,
            "override_outcome": "win"
        }
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.post(url, headers=headers, data=json.dumps(data))
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    override_bet_outcome()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn override_bet_outcome() {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/override-outcome", base_url);
        let client = Client::new();
        let data = json!({
            "bet_id": 123,
            "override_outcome": "win"
        });

        let response = client.post(&url)
            .header("Content-Type", "application/json")
            .json(&data)
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    override_bet_outcome();
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem  value="Success">

    Http Code: `200`
    ```json
    {
        "message": "Bet outcome override updated successfully"
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

    **Bet not found** <br/>
    Http Code: `404`
    ```json
    {
        "status": 404,
        "error": "Bet with ID 123 does not exist"
    }
    ```

    **Wager already decided** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Cannot override bet outcome - wager has already been decided"
    }
    ```

    **Bet outcome already overridden** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Cannot override bet outcome - bet outcome has already been overridden"
    }
    ```

    **Bet offer status not accepted** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Cannot override bet outcome - bet offer status must be 'accepted'"
    }
    ```

    **Bet is inactive** <br/>
    Http Code: `400`
    ```json
    {
        "status": 400,
        "error": "Cannot override bet outcome - bet is inactive"
    }
    ```

    **Validation error** <br/>
    Http Code: `400`
    ```json
    {
        "message": [
            "bet_id must be a number conforming to the specified constraints",
            "override_outcome should not be empty",
            "($value) not a valid bet outcome (undecided, win, loss, push, half-win, half-loss, void)"
        ],
        "error": "Bad Request",
        "statusCode": 400
    }
    ```

  </TabItem>
</Tabs>

## Batch Bet Operations (Atomic)

:::info
This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details.
:::

Execute multiple bet operations in a single atomic transaction. If any operation fails, the entire batch is rolled back and NO operation is applied.

### Request

| Property     | Value                 |
| ------------ | --------------------- |
| method       | `POST`                |
| url          | `$baseUrl/bets/batch` |
| Content-Type | `application/json`    |

### Atomic Behavior

- All operations are validated first (structure, uniqueness, user existence, odds, exposure).
- Exposure / balance checks are performed considering the cumulative effect of the whole batch.
- If ANY operation fails validation or business logic, the entire batch aborts (no partial updates).
- A successful batch returns HTTP `201`.
- A failed batch returns HTTP `400` (or other appropriate error) with no side-effects.
- There is NO HTTP 207 (partial success) for this endpoint.

#### Body

| Field       | Type                               | Required | Description |
| ----------- | ---------------------------------- | -------- | ----------- |
| operations  | Array&lt;BatchOperation&gt;        | Yes      | Ordered list of operations to execute atomically |

#### BatchOperation Type

```typescript
type BatchOperation = {
  operation_type: "make_bet_offer" | "accept_bet_offer" | "cancel_bet_offer" | "update_wager_outcome" | "override_bet_outcome";
  operation_id: string;
  data: object; // Corresponds to the request body of the individual endpoint
}
```

| Field           | Type     | Required | Description |
| --------------- | -------- | -------- | ----------- |
| operation_type  | string   | Yes      | One of allowed enum values (below) |
| operation_id    | string   | Yes      | Client-supplied unique identifier within the batch |
| data            | object   | Yes      | Operation payload - corresponds to the request body of the individual endpoint for that operation type |

:::note Data Field Correspondence

The `data` object for each operation type corresponds exactly to the request body parameters of the individual API endpoints:

- `make_bet_offer` → Uses the same fields as [Make Bet Offer](#make-bet-offer) request body
- `accept_bet_offer` → Uses the same fields as [Accept Bet Offer](#accept-bet-offer) request body  
- `cancel_bet_offer` → Uses the same fields as [Cancel Bet Offer](#cancel-bet-offer) request body
- `update_wager_outcome` → Uses the same fields as [Update Wager Outcome](#update-wager-outcome) request body
- `override_bet_outcome` → Uses the same fields as [Override Bet Outcome](#override-bet-outcome) request body

:::

Duplicate `operation_id` values cause the entire batch to fail.

#### Supported operation_type Values

| Enum Value              | Description |
| ----------------------- | ----------- |
| `make_bet_offer`        | Create a new bet offer (requesting state) |
| `accept_bet_offer`      | Accept (match) existing requesting offers |
| `cancel_bet_offer`      | Cancel open bet offers (requesting state) |
| `update_wager_outcome`  | Decide a wager's outcome |
| `override_bet_outcome`  | Override an accepted bet's outcome |

---

### Operation Payload Schemas (data)

#### 1. make_bet_offer

| Field                      | Type            | Required | Notes |
| -------------------------- | --------------- | -------- | ----- |
| requesting_user_id         | number          | No       | Required if `requesting_user_reference` absent |
| requesting_user_reference  | string          | No       | Required if `requesting_user_id` absent |
| wager_reference            | string          | No       | Required if `bet_id` absent |
| bet_id                     | number          | No       | Alternative to `wager_reference` (takes precedence if both) |
| requesting_odds            | number (>=1)    | Yes      | Decimal odds |
| requesting_amount          | number (>0)     | Yes      | Stake amount |
| meta                       | object          | No       | Arbitrary metadata (string:string) |

#### 2. accept_bet_offer

| Field                      | Type         | Required | Notes |
| -------------------------- | ------------ | -------- | ----- |
| accepting_user_id          | number       | No       | Required if `accepting_user_reference` absent |
| accepting_user_reference   | string       | No       | Required if `accepting_user_id` absent |
| requesting_user_id         | number       | No       | Optional filter (only accept from this user id) |
| requesting_user_reference  | string       | No       | Optional filter (only accept from this user reference) |
| wager_reference            | string       | No       | Required if `bet_id` absent |
| bet_id                     | number       | No       | Alternative to `wager_reference` (takes precedence) |
| maximum_odds               | number (>=1) | Yes      | Reject if matched odds exceed this |
| accepting_amount           | number (>0)  | Yes      | Amount to accept |
| meta                       | object       | No       | Arbitrary metadata |

#### 3. cancel_bet_offer

| Field                      | Type         | Required | Notes |
| -------------------------- | ------------ | -------- | ----- |
| requesting_user_id         | number       | No       | Required if `requesting_user_reference` absent |
| requesting_user_reference  | string       | No       | Required if `requesting_user_id` absent |
| wager_references           | string[]     | No       | Required if `bet_id` absent |
| bet_id                     | number       | No       | Targets one specific bet (odds filters ignored) |
| cancel_amount              | number (>0)  | No       | Partial cancellation amount |
| cancel_all                 | boolean      | No       | If true, cancels all matching amounts |
| minimum_odds               | number (>=1.01) | No    | Only cancel odds &gt;= value (ignored if bet_id) |
| maximum_odds               | number (>=1.01) | No    | Only cancel odds &lt;= value (ignored if bet_id) |

Constraints:
- Provide either `cancel_amount` OR `cancel_all: true`.
- Provide `wager_references` or `bet_id`.
- Only requesting (open) offers are cancelable.
- Partial cancellation leaves remaining amount as a new or adjusted open bet.

#### 4. update_wager_outcome

| Field     | Type                                                                                | Required | Notes |
| --------- | ----------------------------------------------------------------------------------- | -------- | ----- |
| reference | string                                                                              | Yes      | Wager reference |
| outcome   | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void"     | Yes      | New outcome |

#### 5. override_bet_outcome

| Field            | Type                                                                                | Required | Notes |
| ---------------- | ----------------------------------------------------------------------------------- | -------- | ----- |
| bet_id           | number                                                                              | Yes      | Accepted bet id |
| override_outcome | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void"     | Yes      | Override outcome |

:::note Operation Requirements

- **Make Offer**: Either `requesting_user_id` or `requesting_user_reference` required
- **Accept Offer**: Either `accepting_user_id` or `accepting_user_reference` required
- **Cancel Bet Offer**: `requesting_user_id` and `wager_references` are required, plus either `cancel_amount` or `cancel_all`
- **Update Wager Outcome**: `reference` and `outcome` are both required
- **Override Bet Outcome**: `bet_id` and `override_outcome` are both required

:::

:::warning Batch Operation Notes

- **Atomic guarantee**: zero side-effects on any failure.
- **Exposure computation**: Cumulative across all operations in batch
- **Unique operation IDs**: Must be unique within the batch
- **Sequential processing**: Operations processed in array order
- **Validation**: Each operation validated separately before execution
- **Error isolation**: Any single failure aborts entire batch

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/bets/batch" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer YOUR_TOKEN" \
      -d '{
        "operations": [
          {
            "operation_type": "make_bet_offer",
            "operation_id": "offer-1",
            "data": {
              "requesting_user_id": 101,
              "requesting_odds": 2.0,
              "requesting_amount": 100,
              "wager_reference": "atomic-wager-1"
            }
          },
          {
            "operation_type": "accept_bet_offer",
            "operation_id": "accept-1",
            "data": {
              "accepting_user_id": 102,
              "wager_reference": "atomic-wager-1",
              "maximum_odds": 2.2,
              "accepting_amount": 50
            }
          },
          {
            "operation_type": "cancel_bet_offer",
            "operation_id": "cancel-1",
            "data": {
              "requesting_user_id": 101,
              "wager_references": ["atomic-wager-1"],
              "cancel_amount": 25,
              "minimum_odds": 1.8
            }
          },
          {
            "operation_type": "update_wager_outcome",
            "operation_id": "decide-1",
            "data": {
              "reference": "atomic-wager-1",
              "outcome": "win"
            }
          }
        ]
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    async function runAtomicBatch() {
      const body = {
        operations: [
          {
            operation_type: "make_bet_offer",
            operation_id: "offer-1",
            data: {
              requesting_user_id: 101,
              requesting_odds: 2.0,
              requesting_amount: 100,
              wager_reference: "atomic-wager-1"
            }
          },
          {
            operation_type: "accept_bet_offer",
            operation_id: "accept-1",
            data: {
              accepting_user_id: 102,
              wager_reference: "atomic-wager-1",
              maximum_odds: 2.2,
              accepting_amount: 50
            }
          },
          {
            operation_type: "cancel_bet_offer",
            operation_id: "cancel-1",
            data: {
              requesting_user_id: 101,
              wager_references: ["atomic-wager-1"],
              cancel_all: true
            }
          }
        ]
      };

      try {
        const response = await fetch(`${baseUrl}/bets/batch`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer YOUR_TOKEN'
          },
          body: JSON.stringify(body)
        });

        const result = await response.json();
        
        if (response.status === 201) {
          console.log('Batch applied atomically:', result);
        } else {
          console.warn('Batch failed (no changes applied):', result);
        }
      } catch (error) {
        console.error('Error:', error);
      }
    }

    // Example usage
    runAtomicBatch();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json
    from typing import List, Dict, Any

    def run_atomic_batch():
        url = f"{base_url}/bets/batch"
        
        operations = [
            {
                "operation_type": "make_bet_offer",
                "operation_id": "offer-1",
                "data": {
                    "requesting_user_id": 101,
                    "requesting_odds": 2.0,
                    "requesting_amount": 100,
                    "wager_reference": "atomic-wager-1"
                }
            },
            {
                "operation_type": "accept_bet_offer",
                "operation_id": "accept-1",
                "data": {
                    "accepting_user_id": 102,
                    "wager_reference": "atomic-wager-1",
                    "maximum_odds": 2.2,
                    "accepting_amount": 50
                }
            },
            {
                "operation_type": "cancel_bet_offer",
                "operation_id": "cancel-1",
                "data": {
                    "requesting_user_id": 101,
                    "wager_references": ["atomic-wager-1"],
                    "cancel_all": True
                }
            }
        ]
        
        headers = {
            "Content-Type": "application/json",
            "Authorization": "Bearer YOUR_TOKEN"
        }
        
        data = {"operations": operations}
        
        try:
            response = requests.post(url, headers=headers, data=json.dumps(data))
            result = response.json()
            
            if response.status_code == 201:
                print("Batch applied atomically:", result)
            else:
                print("Batch failed (no changes applied):", result)
                
        except Exception as error:
            print("Error:", error)

    # Example usage
    run_atomic_batch()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn run_atomic_batch() {
        let base_url = "your_base_url_here";
        let client = Client::new();

        let payload = json!({
            "operations": [
                {
                    "operation_type": "make_bet_offer",
                    "operation_id": "offer-1",
                    "data": {
                        "requesting_user_id": 101,
                        "requesting_odds": 2.0,
                        "requesting_amount": 100,
                        "wager_reference": "atomic-wager-1"
                    }
                },
                {
                    "operation_type": "accept_bet_offer",
                    "operation_id": "accept-1",
                    "data": {
                        "accepting_user_id": 102,
                        "wager_reference": "atomic-wager-1",
                        "maximum_odds": 2.2,
                        "accepting_amount": 50
                    }
                },
                {
                    "operation_type": "cancel_bet_offer",
                    "operation_id": "cancel-1",
                    "data": {
                        "requesting_user_id": 101,
                        "wager_references": ["atomic-wager-1"],
                        "cancel_all": true
                    }
                }
            ]
        });

        match client.post(format!("{}/bets/batch", base_url))
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer YOUR_TOKEN")
            .json(&payload)
            .send()
            .await {
            Ok(resp) => {
                let status = resp.status();
                let body: serde_json::Value = resp.json().await.unwrap_or(json!({"parse":"error"}));
                println!("Status: {status}");
                println!("Body: {body}");
            },
            Err(e) => eprintln!("Request error: {e}")
        }
    }
    ```

  </TabItem>
</Tabs>

### Response

<Tabs>
  <TabItem value="Success">

    **All operations successful**<br/>
    Http Code: `201`
    ```json
    {
      "message": "All batch operations completed successfully",
      "results": [
        {
          "operation_id": "offer-1",
          "operation_type": "make_bet_offer",
          "success": true,
          "data": {
            "bet_id": 1201,
            "wager_reference": "atomic-wager-1",
            "wager_id": 501
          }
        },
        {
          "operation_id": "accept-1",
          "operation_type": "accept_bet_offer",
          "success": true,
          "data": [
            {
              "bet_id": 1201,
              "requesting_user_id": 101,
              "accepted_amount": 50,
              "accepted_odds": 2.0,
              "wager_reference": "atomic-wager-1",
              "wager_id": 501
            }
          ]
        },
        {
          "operation_id": "cancel-1",
          "operation_type": "cancel_bet_offer",
          "success": true,
          "data": [
            {
              "bet_id": 1203,
              "wager_reference": "atomic-wager-1",
              "wager_id": 501
            }
          ]
        },
        {
          "operation_id": "decide-1",
          "operation_type": "update_wager_outcome",
          "success": true,
          "message": "Wager outcome updated successfully"
        }
      ]
    }
    ```

  </TabItem>
  <TabItem value="Error">

    **Exposure / Balance Aggregation Failure**<br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Batch operations would cause negative effective balance for user(s): 1",
      "data": {
        "balance_issues": [
          {
            "id": 1,
            "balance": 109700,
            "exposure": 2600,
            "effective_balance": 107100,
            "additional_exposure": 10000000000000,
            "preferences": {}
          }
        ],
        "operation_count": 2,
        "affected_users": 1
      }
    }
    ```

    **Duplicate Operation IDs**<br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Operation IDs must be unique within a batch"
    }
    ```

    **Invalid operation_type**<br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Invalid operation type 'invalid_op'. Supported types: make_bet_offer, accept_bet_offer, cancel_bet_offer, update_wager_outcome, override_bet_outcome"
    }
    ```

    **Structural Validation Error**<br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "operations must be a non-empty array"
    }
    ```

    **Per-Operation Validation Failure**<br/>
    Http Code: `400`
    ```json
    {
      "status": 400,
      "error": "Validation failed for batch operations",
      "data": {
        "errors": [
          {
            "operation_id": "offer-1",
            "errors": [
              "requesting_odds must not be less than 1",
              "requesting_amount must be a positive number"
            ]
          },
          {
            "operation_id": "cancel-1",
            "errors": [
              "Either cancel_all must be true or cancel_amount must be provided"
            ]
          }
        ]
      }
    }
    ```

    **Unauthorized**<br/>
    Http Code: `401`
    ```json
    {
      "message": "Unauthorized"
    }
    ```

  </TabItem>
</Tabs>

### Common Failure Reasons

| Scenario                         | Error Snippet |
| -------------------------------- | ------------- |
| Over-exposure                    | "would cause negative effective balance" |
| Duplicate IDs                    | "Operation IDs must be unique within a batch" |
| Missing user identifiers         | "must be defined" |
| Invalid odds/value               | "must not be less than" |
| Nothing to cancel                | "No cancelable bets found" |
| Wager already decided            | "Wager has already been decided" |
| Override invalid state           | "bet offer status must be 'accepted'" |

:::tip Performance Benefits

Batch operations provide several advantages:

- **Reduced Network Overhead**: Multiple operations in a single HTTP request
- **Atomic Transaction Processing**: All-or-nothing guarantee maintains data consistency
- **Cumulative Exposure Validation**: Balance checks consider the entire batch effect
- **Better Error Management**: Single point of failure with detailed error information
- **Improved Throughput**: Ideal for high-frequency trading and bulk operations

:::

:::warning Important Notes

- **Atomic guarantee**: zero side-effects on any failure.
- **No partial success**: Unlike some batch APIs, this endpoint never applies partial results
- **Exposure is computed cumulatively**: multiple make/accept operations may jointly exceed limits.
- **Use unique, meaningful `operation_id` values** for reliable correlation.
- **`bet_id` overrides `wager_reference`** when both are present (in supported operations).
- **Cancel logic ignores odds filters** when `bet_id` is supplied.

:::

## Get Single Bet

:::info

This request must have the authorization header. Refer to [Authorization method](/docs/guides/authentication#authentication-methods) guide for more details

:::

### Request

| Property     | Value                       |
| :----------- | :-------------------------- |
| method       | `GET`                       |
| url          | `$baseUrl/bets/:id`         |
| Content-Type | `application/json`          |

#### Query Params

| Property                        | Type    | Required | Default | Description                      |
| ------------------------------- | ------- | -------- | ------- | -------------------------------- |
| include_bet_trails              | boolean | No       | false   | Include bet trails in response   |
| include_bet_trails_transactions | boolean | No       | false   | Include bet trail transactions   |

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X GET "$baseUrl/bets/123" \
      -H "Content-Type: application/json"
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function getSingleBet(betId) {
      const url = `${baseUrl}/bets/${betId}`;

      fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then(response => response.json())
      .then(data => console.log('Success:', data))
      .catch(error => console.error('Error:', error));
    }

    // Example usage
    getSingleBet(123);
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests

    def get_single_bet(bet_id):
        url = f"{base_url}/bets/{bet_id}"
        headers = {
            "Content-Type": "application/json"
        }

        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            print('Success:', response.json())
        else:
            print('Error:', response.text)

    # Example usage
    get_single_bet(123)
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use tokio;

    #[tokio::main]
    async fn get_single_bet(bet_id: i32) {
        let base_url = "your_base_url_here";
        let url = format!("{}/bets/{}", base_url, bet_id);
        let client = Client::new();

        let response = client.get(&url)
            .header("Content-Type", "application/json")
            .send()
            .await;

        match response {
            Ok(resp) => {
                if resp.status().is_success() {
                    let json: serde_json::Value = resp.json().await.unwrap();
                    println!("Success: {:?}", json);
                } else {
                    let text = resp.text().await.unwrap();
                    println!("Error: {}", text);
                }
            },
            Err(err) => {
                println!("Error: {}", err);
            }
        }
    }

    // Example usage
    get_single_bet(123);
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
            "id": 2,
            "requesting_user_id": 4,
            "requesting_user_reference": "a4_user_2YRDqo69dO4DGdin",
            "accepting_user_id": 4,
            "accepting_user_reference": "a4_user_2YRDqo69dO4DGdin",
            "offer_status": "accepted",
            "effective_amount": 300,
            "effective_odds": 3,
            "override_outcome": null,
            "is_active": true,
            "created_at": "2025-03-11T15:50:33.601Z",
            "wager": {
                "id": 2,
                "reference": "wager-4",
                "outcome": "undecided",
                "account_id": 4,
                "created_at": "2025-03-11T15:50:33.601Z",
                "updated_at": "2025-03-11T15:50:33.601Z"
            },
            "bet_trails": [
                {
                    "id": 4,
                    "bet_id": 2,
                    "account_id": 4,
                    "wager_id": 2,
                    "offer_status": "requesting",
                    "description": "Requesting for bet",
                    "outcome": "undecided",
                    "override_outcome": null,
                    "requesting_odds": 3,
                    "requesting_amount": 300,
                    "accepting_odds": null,
                    "accepting_amount": null,
                    "effective_odds": null,
                    "effective_amount": null,
                    "created_at": "2025-03-11T15:50:33.601Z",
                    "transactions": []
                },
                {
                    "id": 6,
                    "bet_id": 2,
                    "account_id": 4,
                    "wager_id": 2,
                    "offer_status": "accepted",
                    "description": "Bet offer accepted",
                    "outcome": "undecided",
                    "override_outcome": null,
                    "requesting_odds": 3,
                    "requesting_amount": 300,
                    "accepting_odds": 3,
                    "accepting_amount": 300,
                    "effective_odds": null,
                    "effective_amount": null,
                    "created_at": "2025-03-11T16:20:43.304Z",
                    "transactions": []
                }
            ]
        },
        "message": "Bet retrieved successfully"
    }
    ```

:::info Field Description

**is_active**: A boolean field indicating whether the bet is currently active or inactive. Active bets (true) can be accepted by other users and appear in open bet listings. Inactive bets (false) cannot be accepted and are excluded from open bet results.

:::

  </TabItem>
  <TabItem  value="Error">

    **Unauthorized** <br/>
    Http Code: `401`
    ```json
    {
        "message": "Unauthorized"
    }
    ```

    **Not found** <br/>
    Http Code: `404`
    ```json
    {
        "error": "Bet not found"
    }
    ```

  </TabItem>
</Tabs>
