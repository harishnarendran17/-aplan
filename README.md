-- Step 1: Store subnet range
WITH subnet_data AS (
    SELECT head_int, tail_int, (tail_int - head_int + 1) AS total_range
    FROM ng_inam.subnet
    WHERE subnet_id = 1677
)
-- Step 2: Store assignment table data in range
SELECT DISTINCT cidr, head_int, tail_int
INTO TEMP assignment_temp
FROM ng_inam.assignment
WHERE head_int >= (SELECT head_int FROM subnet_data)
  AND tail_int <= (SELECT tail_int FROM subnet_data);

-- Step 3: Store backbone table data in range
SELECT DISTINCT cidr, head AS head_int, tail AS tail_int
INTO TEMP backbone_temp
FROM ng_inam.ip_audit_backbone_config_feed
WHERE head >= (SELECT head_int FROM subnet_data)
  AND tail <= (SELECT tail_int FROM subnet_data);

-- Step 4: Combine assignment and backbone data, removing duplicates
SELECT cidr, head_int, tail_int
INTO TEMP merged_data
FROM assignment_temp
UNION
SELECT cidr, head_int, tail_int
FROM backbone_temp;

-- Step 5: Calculate assigned range and utilization
WITH range_data AS (
    SELECT 
        (SELECT total_range FROM subnet_data) AS total_range,
        SUM(tail_int - head_int + 1) AS assigned_range
    FROM merged_data
)
UPDATE ng_inam.subnet
SET utilization = (rd.assigned_range::float / rd.total_range::float) * 100
FROM range_data rd
WHERE subnet_id = 1677;
