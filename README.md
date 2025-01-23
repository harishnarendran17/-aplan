WITH filtered_assignment AS (
    SELECT DISTINCT head_int, tail_int
    FROM ng_inam.assignment
    WHERE head_int BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
      AND tail_int BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
),
filtered_backbone AS (
    SELECT DISTINCT head_int, tail_int
    FROM ng_inam.ip_audit_backbone_config_feed
    WHERE head BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
      AND tail BETWEEN 
        (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
        AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
),
combined_ranges AS (
    SELECT DISTINCT head_int, tail_int
    FROM filtered_assignment
    UNION
    SELECT DISTINCT head_int, tail_int
    FROM filtered_backbone
)
SELECT 
    (SUM(tail_int - head_int + 1) * 100.0) / 
    (SELECT (tail_int - head_int + 1) 
     FROM ng_inam.subnet WHERE subnet_id = 1677) AS utilization_percentage
FROM combined_ranges;
