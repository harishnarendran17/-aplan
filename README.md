SELECT *
FROM ng_inam.assignment
WHERE head_int BETWEEN 
    (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
    AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677)
  AND tail_int BETWEEN 
    (SELECT head_int FROM ng_inam.subnet WHERE subnet_id = 1677) 
    AND (SELECT tail_int FROM ng_inam.subnet WHERE subnet_id = 1677);
