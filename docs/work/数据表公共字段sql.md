##### 审批公共字段
	ALTER TABLE "qmys"."budget_adjust" 
		ADD COLUMN "approval_node_id" text,
		ADD COLUMN "current_approvers" text,
		ADD COLUMN "approval_status" text,
	  ADD COLUMN "approvers" text,
	  ADD COLUMN "cc_person" text;
	
	COMMENT ON COLUMN "qmys"."budget_adjust"."approval_node_id" IS '当前审批节点id';
	
	COMMENT ON COLUMN "qmys"."budget_adjust"."current_approvers" IS '当前审批人';
	
	COMMENT ON COLUMN "qmys"."budget_adjust"."approval_status" IS '审核状态';
	
	COMMENT ON COLUMN "qmys"."budget_adjust"."approvers" IS '所有审批过的人';
	
	COMMENT ON COLUMN "qmys"."budget_adjust"."cc_person" IS '抄送人';

