# Transaction Management and Concurrency Control
## Answers

---

## 6a. Purchase Transaction

```sql
BEGIN TRANSACTION;

INSERT INTO INVOICE (INV_NUMBER, CUS_CODE, INV_DATE, INV_TOTAL, INV_TERMS, INV_STATUS)
VALUES (10983, 10010, '2018-05-11', 118.80, 30, 'OPEN');

INSERT INTO LINE (INV_NUMBER, LINE_NUMBER, P_CODE, LINE_UNITS, LINE_PRICE)
VALUES (10983, 1, '11QER/31', 1, 110.00);

UPDATE PRODUCT
SET P_QTYOH = P_QTYOH - 1
WHERE P_CODE = '11QER/31';

UPDATE CUSTOMER
SET CUS_BALANCE = CUS_BALANCE + 118.80
WHERE CUS_CODE = 10010;

UPDATE CUSTOMER
SET CUS_DATESTPUR = '2018-05-11'
WHERE CUS_CODE = 10010;

COMMIT;
```

---

## 6b. Payment Transaction

```sql
BEGIN TRANSACTION;

INSERT INTO PAYMENTS (PMT_ID, PMT_DATE, CUS_CODE, PMT_AMT, PMT_TYPE, PMT_DETAILS)
VALUES (3428, '2018-06-03', 10010, 100.00, 'CASH', NULL);

UPDATE CUSTOMER
SET CUS_BALANCE = CUS_BALANCE - 100.00
WHERE CUS_CODE = 10010;

UPDATE CUSTOMER
SET CUS_DATESTLPMT = '2018-06-03'
WHERE CUS_CODE = 10010;

COMMIT;
```

---

## 7. Create a Transaction Log

The following table shows a simple chronological transaction log for
Transaction 6a (purchase) and Transaction 6b (payment), adapted from Table 10.14.

| Step | Transaction | Operation | Object | Description |
|-----|------------|-----------|--------|-------------|
| 1 | T6a (Purchase) | BEGIN | TRANSACTION | Purchase transaction starts |
| 2 | T6a | INSERT | INVOICE | New invoice (INV_NUMBER = 10983) is created |
| 3 | T6a | INSERT | LINE | Invoice line added for product 11QER/31 |
| 4 | T6a | READ | PRODUCT | Current P_QTYOH is read |
| 5 | T6a | UPDATE | PRODUCT | P_QTYOH reduced by 1 |
| 6 | T6a | READ | CUSTOMER | Current CUS_BALANCE is read |
| 7 | T6a | UPDATE | CUSTOMER | CUS_BALANCE increased by 118.80 |
| 8 | T6a | UPDATE | CUSTOMER | CUS_DATESTPUR set to 2018-05-11 |
| 9 | T6a | COMMIT | TRANSACTION | Purchase transaction committed |
| 10 | T6b (Payment) | BEGIN | TRANSACTION | Payment transaction starts |
| 11 | T6b | INSERT | PAYMENTS | New cash payment (PMT_ID = 3428) inserted |
| 12 | T6b | READ | CUSTOMER | Current CUS_BALANCE is read |
| 13 | T6b | UPDATE | CUSTOMER | CUS_BALANCE decreased by 100.00 |
| 14 | T6b | UPDATE | CUSTOMER | CUS_DATESTLPMT set to 2018-06-03 |
| 15 | T6b | COMMIT | TRANSACTION | Payment transaction committed |

---

## 8. Pessimistic Locking Analysis (2PL NOT used)

**Assumption:**  
ABC Markets uses pessimistic locking, but the two-phase locking protocol (2PL) is NOT enforced.

### Chronological operation list for Transaction 6a (Purchase)

1. Transaction T6a begins (`BEGIN TRANSACTION`).
2. T6a requests an exclusive lock on CUSTOMER (CUS_CODE = 10010); lock granted.
3. T6a reads CUSTOMER balance.
4. T6a updates CUSTOMER balance and last purchase date.
5. CUSTOMER lock is released before COMMIT.
6. T6a requests an exclusive lock on PRODUCT (P_CODE = '11QER/31'); lock granted.
7. T6a reads PRODUCT quantity.
8. T6a updates PRODUCT quantity.
9. PRODUCT lock is released before COMMIT.
10. T6a inserts INVOICE.
11. INVOICE lock is released before COMMIT.
12. T6a inserts LINE.
13. LINE lock is released before COMMIT.
14. T6a commits.

### Potential anomalies caused by lack of 2PL

- **Dirty Read**
- **Lost Update**
- **Inconsistent State**
- **Non-repeatable Reads**
