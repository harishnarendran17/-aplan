-- Step 1: Get the total range for the subnet
WITH subnet_data AS (
    SELECT head_int, tail_int, (tail_int - head_int + 1) AS total_range
    FROM ng_inam.subnet
    WHERE subnet_id = 1677
),

-- Step 2: Fetch data from the assignment table within the subnet range
assignment_data AS (
    SELECT DISTINCT cidr, head_int, tail_int
    FROM ng_inam.assignment
    WHERE head_int >= (SELECT head_int FROM subnet_data)
      AND tail_int <= (SELECT tail_int FROM subnet_data)
),

-- Step 3: Fetch data from the backbone table within the subnet range
backbone_data AS (
    SELECT DISTINCT cidr, head, tail
    FROM ng_inam.ip_audit_backbone_config_feed
    WHERE head >= (SELECT head_int FROM subnet_data)
      AND tail <= (SELECT tail_int FROM subnet_data)
),

-- Step 4: Combine assignment and backbone data, remove duplicates, and add unique rows
merged_data AS (
    SELECT cidr, head_int AS head, tail_int AS tail
    FROM assignment_data
    UNION
    SELECT cidr, head, tail
    FROM backbone_data
),

-- Step 5: Calculate assigned range from the merged data
range_data AS (
    SELECT 
        (SELECT total_range FROM subnet_data) AS total_range,
        SUM(tail - head + 1) AS assigned_range
    FROM merged_data
)

-- Step 6: Update the utilization in the subnet table
UPDATE ng_inam.subnet
SET utilization = (rd.assigned_range::float / rd.total_range::float) * 100
FROM range_data rd
WHERE subnet_id = 1677;
