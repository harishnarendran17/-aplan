WITH RECURSIVE parent AS (
    SELECT 
        subnet_id, 
        parent_id, 
        name
    FROM 
        ng_inam.subnet
    WHERE 
        subnet_id = 1677
    UNION
    SELECT 
        ts.subnet_id, 
        ts.parent_id, 
        ts.name
    FROM 
        ng_inam.subnet ts
    JOIN 
        parent p 
    ON 
        p.subnet_id = ts.parent_id
),
subnet_ids AS (
    SELECT 
        subnet_id 
    FROM 
        parent
)
UPDATE ng_inam.subnet s
SET utilization = (
    (
        SELECT 
            COALESCE(SUM(a.tail_int - a.head_int + 1), 0) AS value
        FROM 
            ng_inam.assignment a
        WHERE 
            a.subnet_id IN (SELECT subnet_id FROM subnet_ids)
    ) / (s.tail_int - s.head_int + 1)
) * 100
WHERE 
    s.subnet_id IN (SELECT subnet_id FROM subnet_ids);

    
