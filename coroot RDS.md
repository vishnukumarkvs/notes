coroot RDS
==========

Any PostgreSQL LOGIN role can be IAM-enabled – coroot_iam, coroot, or even root/postgres master – as long as you GRANT rds_iam TO <user>.
But don't use root/master:
- Master has rds_superuser → if token leaks, attacker has superuser. IAM still checks IAM policy, but blast radius too large.
- Best practice: create dedicated low-privilege user coroot_iam (or coroot) with only CONNECT + Coroot schema privileges.
RDS vs Aurora – same SQL, must be done after instance is up:
IAM flag Enable IAM auth is at instance creation (Terraform iam_database_authentication_enabled = true), but the DB role is not auto-created – you must run SQL once as master:
-- connect as master (postgres/root) via password once
CREATE USER coroot_iam WITH LOGIN;
GRANT rds_iam TO coroot_iam;
GRANT CONNECT ON DATABASE coroot TO coroot_iam;
GRANT ALL ON SCHEMA public TO coroot_iam; -- or your Coroot schema
-- Aurora: run on writer, replicates to readers automatically
When to create:
- Terraform: aws_db_instance + null_resource/aws_lambda provisioner or terraform-data that psql as master, or via init SQL in a Job. Not at CREATE DATABASE – RDS doesn't support init scripts like CloudNativePG.
- Manual/CI: psql "host=... user=master ..." once after terraform apply.
- For Aurora: same, but connect to cluster writer endpoint; GRANT replicates.
Then PG_CONNECTION_STRING user=coroot_iam must exactly match IAM policy resource:
"Resource": "arn:aws:rds-db:us-east-1:123456789012:dbuser:db-ABC123/coroot_iam"
So coroot_iam is just an example – use whatever name you create, but keep it dedicated, create at provisioning post-step, and map it 1:1 to the IAM policy. Using coroot is fine too if you already have that user – just GRANT rds_iam TO coroot; and switch PG_CONNECTION_STRING user=coroot.



