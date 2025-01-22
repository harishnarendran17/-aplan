UPDATE ng_inam.subnet s
SET utilization = (
    (
        SELECT 
            CASE 
                WHEN SUM(used_capacity) IS NOT NULL 
                THEN SUM(used_capacity) 
                ELSE 0 
            END
        FROM (
            -- Calculate utilization from the assignment table
            SELECT 
                a.subnet_id, 
                SUM(a.tail_int - a.head_int + 1) AS used_capacity
            FROM ng_inam.assignment a
            WHERE a.subnet_id IN (
                WITH RECURSIVE parent AS (
                    SELECT subnet_id, parent_id, name 
                    FROM ng_inam.subnet
                    WHERE subnet_id = s.subnet_id
                    UNION ALL
                    SELECT ts.subnet_id, ts.parent_id, ts.name 
                    FROM ng_inam.subnet ts
                    INNER JOIN parent p ON p.subnet_id = ts.parent_id
                )
                SELECT subnet_id FROM parent
            )
            GROUP BY a.subnet_id

            UNION ALL

            -- Calculate utilization from the ip_audit_backbone_config_feed table
            SELECT 
                c.subnet_id, 
                SUM(c.tail - c.head + 1) AS used_capacity
            FROM ng_inam.ip_audit_backbone_config_feed c
            WHERE c.subnet_id IN (
                WITH RECURSIVE parent AS (
                    SELECT subnet_id, parent_id, name 
                    FROM ng_inam.subnet
                    WHERE subnet_id = s.subnet_id
                    UNION ALL
                    SELECT ts.subnet_id, ts.parent_id, ts.name 
                    FROM ng_inam.subnet ts
                    INNER JOIN parent p ON p.subnet_id = ts.parent_id
                )
                SELECT subnet_id FROM parent
            )
            GROUP BY c.subnet_id
        ) AS combined_utilization
    ) / (s.tail_int - s.head_int + 1) * 100
)
WHERE s.subnet_id IN (SELECT subnet_id FROM ng_inam.subnet);
