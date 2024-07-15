WITH AddressValidation AS (
  SELECT 
    customer_id,
    COUNT(CASE WHEN address_type = 'mailing' THEN 1 END) AS mailing_count,
    COUNT(CASE WHEN address_type = 'home' THEN 1 END) AS home_count,
    ARRAY_AGG(address_line1) AS address_lines1,
    ARRAY_AGG(address_line2) AS address_lines2,
    ARRAY_AGG(address_line3) AS address_lines3,
    ARRAY_AGG(address_type) AS address_types
  FROM 
    `your_dataset.your_address_table`
  GROUP BY 
    customer_id
)

SELECT 
  customer_id
FROM 
  AddressValidation
WHERE 
  mailing_count != 1
  OR home_count != 1
  OR EXISTS (
    SELECT 1
    FROM UNNEST(address_lines1) AS addr1
    WHERE addr1 IS NULL OR addr1 NOT REGEXP 'your_regex_for_address_line1'
  )
  OR EXISTS (
    SELECT 1
    FROM UNNEST(address_lines2) AS addr2
    WHERE addr2 IS NOT NULL AND addr2 NOT REGEXP 'your_regex_for_address_line2'
  )
  OR EXISTS (
    SELECT 1
    FROM UNNEST(address_lines3) AS addr3
    WHERE addr3 IS NOT NULL
  )
