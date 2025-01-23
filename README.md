WITH merged_ranges AS (
    SELECT DISTINCT ON (cidr) 
        cidr, head_int, tail_int
    FROM (
        SELECT cidr, head_int, tail_int FROM assignment_temp
        UNION ALL
        SELECT cidr, head_int, tail_int FROM backbone_temp
    ) AS combined_data
    ORDER BY cidr, head_int, tail_int
)
SELECT * 
INTO TEMP merged_data
FROM merged_ranges;
