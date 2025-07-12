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

| Property                  | Type         | Required | Default | Description               |
| ------------------------- | ------------ | -------- | ------- | ------------------------- |
| wager_reference           | string       | Yes      | -       | Wager reference           |
| requesting_odds           | number (>=1) | Yes      | -       | Requesting odds           |
| requesting_amount         | number (>0)  | Yes      | -       | Requesting amount         |
| requesting_user_id        | number       | No       | -       | Requesting user id        |
| requesting_user_reference | number       | No       | -       | Requesting user reference |

:::info

Either the `requesting_user_id` or `requesting_user_reference` must be provided

:::

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/bets/make-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "wager_reference": "example_wager_reference",
        "requesting_odds": 2,
        "requesting_amount": 100,
        "requesting_user_id": 12345
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function makeBetOffer() {
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

    // Example usage
    makeBetOffer();
    ```

  </TabItem>
  <TabItem value="python" label="Python">

    ```python
    import requests
    import json

    def make_bet_offer():
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

    # Example usage
    make_bet_offer()
    ```

  </TabItem>
  <TabItem value="rust" label="Rust">

    ```rust
    use reqwest::Client;
    use serde_json::json;
    use tokio;

    #[tokio::main]
    async fn make_bet_offer() {
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

    // Example usage
    make_bet_offer();
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
        "error": "Requesting user not found"
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
| wager_reference           | string       | Yes      | -       | Wager reference           |
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

<Tabs groupId="programming-language">
  <TabItem value="curl" label="cURL">

    ```bash
    curl -X POST "$baseUrl/bets/accept-offer" \
      -H "Content-Type: application/json" \
      -d '{
        "wager_reference": "example_wager_reference",
        "maximum_odds": 2,
        "accepting_amount": 100,
        "accepting_user_id": 12345,
        "meta": {}
      }'
    ```

  </TabItem>
  <TabItem value="javascript" label="JavaScript">

    ```javascript
    function acceptBetOffer() {
      const url = `${baseUrl}/bets/accept-offer`;
      const data = {
        wager_reference: "example_wager_reference",
        maximum_odds: 2,
        accepting_amount: 100,
        accepting_user_id: 12345,
        meta: {}
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
    acceptBetOffer();
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
        "error": "Requesting user not found"
    }
    ```

    **Too much accepted amount** <br/>
    Http Code: `400`
    ```json
    {
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
| requesting_user_ids        | number(comma separated)                                  | No       | -            | Comma separated requesting user ids        |
| requesting_user_references | string(comma separated)                                  | No       | -            | Comma separated requesting user references |
| created_after              | number(timestamp) \| DateString                          | No       | -            | Filter result from this date               |
| created_before             | number(timestamp) \| DateString                          | No       | -            | Filter result to this date                 |
| minimum_odds               | number                                                   | No       | -            | Minimum odds                               |
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

| Property         | Type                                                                            | Required | Default      | Description                      |
| ---------------- | ------------------------------------------------------------------------------- | -------- | ------------ | -------------------------------- |
| page             | number                                                                          | No       | 1            | Current page                     |
| per_page         | number                                                                          | No       | 20           | Number of items per page         |
| wager_references | string                                                                          | No       | -            | Comma separated wager references |
| user_ids         | number(comma separated)                                                         | No       | -            | Comma separated user ids         |
| user_references  | string(comma separated)                                                         | No       | -            | Comma separated user references  |
| bet_outcomes     | "undecided" \| "win" \| "loss" \| "push" \| "half-win" \| "half-loss" \| "void" | No       | -            | Comma separated bet outcomes     |
| created_after    | number(timestamp) \| DateString                                                 | No       | -            | Filter result from this date     |
| created_before   | number(timestamp) \| DateString                                                 | No       | -            | Filter result to this date       |
| sort_by          | "effective_odds" \| "effective_amount" \| "created_at"                          | No       | "created_at" | Sort by                          |
| sort_direction   | "asc" \| "desc"                                                                 | No       | "asc"        | Sort direction                   |

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

    **Wager already decided** <br/>
    Http Code: `400`
    ```json
    {
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
