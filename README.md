WITH subnet_data AS (
    SELECT head_int, tail_int, (tail_int - head_int + 1) AS total_range
    FROM ng_inam.subnet
    WHERE subnet_id = 1677
),
filtered_data AS (
    SELECT head_int, tail_int
    FROM ng_inam.assignment
    WHERE head_int >= (SELECT head_int FROM subnet_data)
      AND tail_int <= (SELECT tail_int FROM subnet_data)
    UNION ALL  -- Efficiently combine assignment and backbone data without removing duplicates
    SELECT head, tail
    FROM ng_inam.ip_audit_backbone_config_feed
    WHERE head >= (SELECT head_int FROM subnet_data)
      AND tail <= (SELECT tail_int FROM subnet_data)
),
range_data AS (
    SELECT 
        (SELECT total_range FROM subnet_data) AS total_range,
        SUM(tail_int - head_int + 1) AS assigned_range
    FROM filtered_data
)
UPDATE ng_inam.subnet
SET utilization = (rd.assigned_range::float / rd.total_range::float) * 100
FROM range_data rd
WHERE subnet_id = 1677;
