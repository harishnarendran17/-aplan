import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface SubnetRepository extends JpaRepository<Subnet, Integer> {

    @Query("select case when exists ( " +
           "  select 1 " +
           "  from ng_inam.subnet_validator sv " +
           "  join ng_inam.validator v on sv.validator_id = v.validator_id " +
           "  where sv.subnet_id = :subnetId " +
           "    and v.validator = 'Backbone Config Validator' " +
           "    and sv.validator_id = 3) " +
           "then true else false end " +
           "from Subnet s where s.subnetId = :subnetId")
    boolean isBackboneEnabled(@Param("subnetId") int subnetId);
}
