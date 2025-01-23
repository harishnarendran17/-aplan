WITH filtered_assignment AS (
    SELECT DISTINCT head_int AS head, tail_int AS tail, cidr
    FROM WITH filtered_data AS (
    SELECT DISTINCT 
        CASE 
            WHEN a.head_int BETWEEN s.head_int AND s.tail_int 
            THEN a.head_int 
            ELSE NULL 
        END AS head, 
        CASE 
            WHEN a.tail_int BETWEEN s.head_int AND s.tail_int 
            THEN a.tail_int 
            ELSE NULL 
        END AS tail, 
        a.cidr
    FROM ng_inam.subnet s
    LEFT JOIN ng_inam.assignment a ON a.head_int BETWEEN s.head_int AND s.tail_int 
                                     AND a.tail_int BETWEEN s.head_int AND s.tail_int
    WHERE s.subnet_id = 1677
    UNION
    SELECT DISTINCT 
        CASE 
            WHEN b.head BETWEEN s.head_int AND s.tail_int 
            THEN b.head 
            ELSE NULL 
        END AS head, 
        CASE 
            WHEN b.tail BETWEEN s.head_int AND s.tail_int 
            THEN b.tail 
            ELSE NULL 
        END AS tail, 
        b.cidr
    FROM ng_inam.subnet s
    LEFT JOIN ng_inam.ip_audit_backbone_config_feed b ON b.head BETWEEN s.head_int AND s.tail_int
                                                     AND b.tail BETWEEN s.head_int AND s.tail_int
    WHERE s.subnet_id = 1677
),
range_data AS (
    SELECT 
        s.subnet_id,
        (s.tail_int - s.head_int + 1) AS total_range,
        SUM(CASE WHEN fd.head IS NOT NULL AND fd.tail IS NOT NULL THEN fd.tail - fd.head + 1 ELSE 0 END) AS assigned_range
    FROM ng_inam.subnet s
    LEFT JOIN filtered_data fd ON fd.head >= s.head_int AND fd.tail <= s.tail_int
    WHERE s.subnet_id = 1677
    GROUP BY s.subnet_id, s.head_int, s.tail_int
)
UPDATE ng_inam.subnet s
SET utilization = (rd.assigned_range::float / rd.total_range::float) * 100
FROM range_data rd
WHERE s.subnet_id = rd.subnet_id;
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
