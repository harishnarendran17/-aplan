WITH filtered_assignment AS (
    SELECT DISTINCT head_int AS head, tail_int AS tail, cidr
    FROM ng_inam.assignment
    WHERE head_int BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
      AND tail_int BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
),
filtered_backbone AS (
    SELECT DISTINCT head, tail, cidr
    FROM ng_inam.ip_audit_backbone_config_feed
    WHERE head BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
      AND tail BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
),
combined_ranges AS (
    SELECT head, tail, cidr FROM filtered_assignment
    UNION
    SELECT head, tail, cidr FROM filtered_backbone
),
merged_ranges AS (
    SELECT DISTINCT 
        head, 
        tail, 
        cidr
    FROM combined_ranges
),
range_data AS (
    SELECT 
        s.subnet_id, 
        (s.tail_int - s.head_int + 1) AS total_range,  
        SUM(mr.tail - mr.head + 1) AS assigned_range  
    FROM ng_inam.subnet s
    LEFT JOIN merged_ranges mr ON mr.head >= s.head_int AND mr.tail <= s.tail_int
    WHERE s.subnet_id = 1677
    GROUP BY s.subnet_id, s.head_int, s.tail_int
)
UPDATE ng_inam.subnet s
SET utilization = (rd.assigned_range::float / rd.total_range::float) * 100
FROM range_data rd
WHERE s.subnet_id = rd.subnet_id;
