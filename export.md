WITH invalid_addresses AS (
  SELECT 
    customer_id,
    COUNTIF(address_type IN ('mailing', 'home')) AS address_count,
    STRING_AGG(address_type) AS address_types,
    STRING_AGG(address_line1) AS address_lines1,
    STRING_AGG(address_line2) AS address_lines2,
    STRING_AGG(address_line3) AS address_lines3,
    ARRAY_AGG(
      CASE 
        WHEN address_line1 IS NULL OR NOT REGEXP_CONTAINS(address_line1, r'^[A-Za-z0-9\s,]+$') THEN 1 
        ELSE 0 
      END
    ) AS invalid_address1,
    ARRAY_AGG(
      CASE 
        WHEN address_line2 IS NOT NULL AND NOT REGEXP_CONTAINS(address_line2, r'^[A-Za-z0-9\s,]+$') THEN 1 
        ELSE 0 
      END
    ) AS invalid_address2,
    ARRAY_AGG(
      CASE 
        WHEN address_line3 IS NOT NULL THEN 1 
        ELSE 0 
      END
    ) AS has_address3
  FROM 
    customer_addresses
  GROUP BY 
    customer_id
)
SELECT 
  customer_id
FROM 
  invalid_addresses
WHERE 
  ARRAY_TO_STRING(invalid_address1, '') CONTAINS '1' OR 
  ARRAY_TO_STRING(invalid_address2, '') CONTAINS '1' OR 
  ARRAY_TO_STRING(has_address3, '') CONTAINS '1' OR
  address_count != 1;
