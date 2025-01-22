UPDATE ng_inam.subnet s
SET utilization = (
    -- Calculate total used capacity from both assignment and config tables
    (
        SELECT 
            COALESCE(SUM(used_capacity), 0)
        FROM (
            -- Used capacity from assignment table
            SELECT 
                SUM(a.tail_int - a.head_int + 1) AS used_capacity
            FROM ng_inam.assignment a
            WHERE a.subnet_id = s.subnet_id
            UNION ALL
            -- Used capacity from config table (only if the subnet is in the config table)
            SELECT 
                SUM(c.tail - c.head + 1) AS used_capacity
            FROM ng_inam.ip_audit_backbone_config_feed c
            WHERE c.cidr = s.cidr
        ) AS combined_utilization
    ) / (s.tail_int - s.head_int + 1) * 100
)
WHERE s.subnet_id = :subnet_id
AND EXISTS (
    -- Check if the subnet_id exists in the config table
    SELECT 1
    FROM ng_inam.ip_audit_backbone_config_feed c
    WHERE c.cidr = s.cidr
);
