UPDATE ng_inam.subnet s
SET utilization = (
    (
        -- Sum used capacity from assignment and config tables
        SELECT 
            COALESCE(SUM(used_capacity), 0)
        FROM (
            -- Calculate used capacity from the assignment table
            SELECT 
                SUM(a.tail_int - a.head_int + 1) AS used_capacity
            FROM ng_inam.assignment a
            WHERE a.subnet_id = s.subnet_id
        UNION ALL
            -- Calculate used capacity from the config table
            SELECT 
                SUM(c.tail - c.head + 1) AS used_capacity
            FROM ng_inam.ip_audit_backbone_config_feed c
            WHERE c.cidr = s.cidr
        ) AS combined_utilization
    ) / (s.tail_int - s.head_int + 1) * 100
)
WHERE s.subnet_id IN (
    SELECT DISTINCT subnet_id FROM ng_inam.assignment
);
