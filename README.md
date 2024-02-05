I have used the bitcoin Data.
**bigquery-public-data.bitcoin_blockchain.transactions**
The bigquery-public-data.bitcoin_blockchain.transactions table in Google BigQuery provides access to historical Bitcoin blockchain transaction data. This can be a valuable resource for anyone interested in analyzing Bitcoin transactions, understanding network activity, or conducting research on various aspects of the cryptocurrency.



**1. What has been the average transaction fee over the past year?**

```SQL
SELECT
  AVG(fee) AS average_fee
FROM `bigquery-public-data.crypto_bitcoin.transactions`
WHERE EXTRACT(YEAR FROM block_timestamp) = EXTRACT(YEAR FROM CURRENT_TIMESTAMP())
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/0bce86d6-5e93-4b9b-ae20-b49d4595ab83)

**2. How many transactions are in the dataset?**

```SQL
SELECT COUNT(*) AS total_transactions
FROM `bigquery-public-data.bitcoin_blockchain.transactions`;
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/02e2c10e-7d51-4bda-b9ca-4e3dee1239ce)

**3. What are the top 2 largest transactions by output value**
```SQL
SELECT transaction_id, outputs
FROM `bigquery-public-data.bitcoin_blockchain.transactions`
ORDER BY transaction_id DESC
LIMIT 2;
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/137f63ab-4be5-43dc-bcf3-8c8c51141b8d)

**4. How many transactions were there per year?**
```SQL
SELECT EXTRACT(YEAR FROM TIMESTAMP_MILLIS(timestamp)) AS year, COUNT(*) AS transaction_count
FROM `bigquery-public-data.bitcoin_blockchain.transactions`
GROUP BY year
ORDER BY year;
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/5581a6c2-dd78-4080-9872-94ba6294a148)

**5. How many unique addresses have received Bitcoin?**
```SQL
SELECT COUNT(DISTINCT output_script_bytes
) as unique_addresses
FROM `bigquery-public-data.bitcoin_blockchain.transactions`, UNNEST(outputs) as outputs;
```

![image](https://github.com/shrest2/Shrestha/assets/159014568/24df4e2a-7cf3-46b9-854e-b1dddcdcc2a9)

**6. What is the distribution of transaction version numbers?**
```SQL
SELECT version, COUNT(*) AS version_count
FROM `bigquery-public-data.bitcoin_blockchain.transactions`
GROUP BY version
ORDER BY version_count DESC;
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/d887f922-91bb-4b2a-b7d1-3cd86c4e4f1d)

**7. Find the block with the highest cumulative transaction value.**
```SQL
SELECT block_id, SUM(outputs.output_satoshis) AS total_block_value
FROM `bigquery-public-data.bitcoin_blockchain.transactions`, UNNEST(outputs) AS outputs
GROUP BY block_id
ORDER BY total_block_value DESC
LIMIT 1;
```
![image](https://github.com/shrest2/Shrestha/assets/159014568/3276a41e-8d99-4fed-ae0b-00e19dbe0cca)

**8. How many transactions have a non-zero work error?**
```SQL
SELECT COUNT(*) AS transactions_with_work_error
FROM `bigquery-public-data.bitcoin_blockchain.transactions`
WHERE work_error != 0;
```

![image](https://github.com/shrest2/Shrestha/assets/159014568/6296980f-a077-4837-a4f8-faf7e8050003)

**9. What are the total inputs and outputs values for transactions in the blocks mined without a nonce in the last 6 months?**
```SQL
SELECT t.block_id, SUM(outputs.value) AS total_output_value, SUM(inputs.value) AS total_input_value
FROM `bigquery-public-data.bitcoin_blockchain.transactions` AS t
LEFT JOIN `bigquery-public-data.bitcoin_blockchain.blocks` AS b ON t.block_id = b.block_id
WHERE b.nonce IS NULL AND t.block_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 6 MONTH)
GROUP BY t.block_id;
```

**10. For transactions in the last week, how does the version number correlate with block timestamp delay?**
```SQL
SELECT t.transaction_id, t.version, TIMESTAMP_DIFF(b.block_timestamp, t.block_timestamp, SECOND) AS timestamp_delay
FROM `bigquery-public-data.bitcoin_blockchain.transactions` AS t
LEFT JOIN `bigquery-public-data.bitcoin_blockchain.blocks` AS b ON t.block_id = b.block_id
WHERE t.block_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 WEEK)
ORDER BY timestamp_delay DESC;
```
