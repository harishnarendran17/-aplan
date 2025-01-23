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
    ORDER BY cidr, head, tail
),
merged_ranges_final AS (
    SELECT 
        cidr,
        MIN(head) AS merged_head,
        MAX(tail) AS merged_tail
    FROM merged_ranges
    GROUP BY cidr
)
SELECT 
    (SUM(merged_ranges_final.merged_tail - merged_ranges_final.merged_head + 1) * 100.0) /
    (SELECT tail_int - head_int + 1 FROM ng_inam.subnet WHERE subnet_id = 1677) AS utilization_percentage
FROM merged_ranges_final;
