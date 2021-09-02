kubectl run ora-test --image=localhost:5000/database-enterprise:12.2.0.1 \
--env=DB_SID=ORCLCDB \
--env=DB_PDB=ORCLPDB \
--env=ORACLE_PWD=dada \
--env=ORACLE_CHARACTERSET=AL32UTF8 \
--port=1521 \
--port=5500