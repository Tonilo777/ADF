# Development Standards & Best Practices

## Core Principles

### 1. Documentation-First Approach
**MANDATORY before any implementation:**
- **Research official vendor documentation** - Always check latest version
- **Verify syntax compatibility** with specific connector/tool versions  
- **Cross-reference limitations** and known issues
- **Flag ALL uncertainties** immediately - never assume

### 2. Production-Ready Priority
- **Primary goal: Production stability** over speed or convenience
- **Test all failure scenarios** before implementation
- **Design for monitoring and troubleshooting**
- **Document all assumptions and dependencies**

## Critical Verification Protocol

### Before ANY Code Changes:

#### Step 1: Documentation Research (MANDATORY)
- [ ] Check official vendor docs for EXACT syntax requirements
- [ ] Verify connector/integration-specific limitations
- [ ] Identify version-specific breaking changes or deprecations
- [ ] Search for known error patterns and solutions

#### Step 2: Syntax Validation
- [ ] Validate SQL syntax against specific database connector documentation
- [ ] Verify column counts, data types, and required clauses
- [ ] Check for connector-specific restrictions (VALUES vs SELECT, etc.)
- [ ] Confirm case sensitivity requirements

#### Step 3: Error Handling Design  
- [ ] Plan for null/empty result scenarios
- [ ] Add appropriate type conversions and validations
- [ ] Include fallback mechanisms for failure cases
- [ ] Design retry logic where appropriate

## Common Production Issues & Solutions

### SQL Syntax Errors

**❌ Wrong: INSERT...VALUES in ADF Script Activity**
```sql
INSERT INTO table VALUES (val1, val2, val3)
```

**✅ Correct: INSERT...SELECT with column aliases**
```sql
INSERT INTO table (col1, col2, col3)
SELECT 
  val1 as col1,
  val2 as col2, 
  val3 as col3;
```

### Data Type Handling

**❌ Wrong: Direct numeric comparison from Snowflake**
```json
"@equals(activity().output.firstRow.COUNT, 0)"
```

**✅ Correct: Type conversion required**
```json
"@equals(int(activity().output.firstRow.COUNT), 0)"
```

### Schema References

**❌ Wrong: Mixed case schema names**
```sql
SELECT * FROM javatest.tonilo_warehouse.dim_tenant
```

**✅ Correct: Consistent uppercase for Snowflake**
```sql
SELECT * FROM JAVATEST.TONILO_WAREHOUSE.DIM_TENANT
```

### Null Handling

**❌ Wrong: No null checks on lookup results**
```json
"@string(activity().output.firstRow.ID)"
```

**✅ Correct: Defensive null handling**
```json
"@if(greater(length(coalesce(string(activity().output.firstRow.ID), '')), 0), string(activity().output.firstRow.ID), '')"
```

## ADF + Snowflake Specific Requirements

### Script Activities
- **NEVER use `FOR UPDATE`** - Not supported
- **NEVER create procedures at runtime** - Impossible in Script activities
- **Always use `INSERT...SELECT`** instead of `INSERT...VALUES`
- **Include column aliases** in SELECT statements for clarity
- **Use proper column count matching** - INSERT expects exact column match

### Lookup Activities  
- **All numeric results are strings** - Always convert with `int()` or `float()`
- **Always handle empty/null results** with defensive checks
- **Use uppercase schema names** for consistency

### Identity Resolution Pattern
**Use 6-step pattern only:**
1. Merge → 2. Lookup → 3. SetVariable (repeat for each dimension)

**Never use atomic/single-step resolution attempts**

## Error Prevention Checklist

Before committing ANY pipeline changes:

- [ ] Researched official documentation for all syntax used
- [ ] Verified connector-specific limitations and requirements  
- [ ] Tested with empty/null data scenarios
- [ ] Validated column counts in all INSERT statements
- [ ] Added proper type conversions for numeric comparisons
- [ ] Used consistent schema name casing
- [ ] Included defensive null/empty checks
- [ ] Followed established architectural patterns
- [ ] Added appropriate error handling and logging

## Red Flags - Stop and Investigate

### Immediate Documentation Research Required:
- Any SQL syntax you haven't used before in this specific connector
- Column count mismatches in INSERT statements  
- Numeric comparison failures
- Unexpected null/empty results
- Case sensitivity issues
- Connector-specific error messages

### Never Assume:
- SQL syntax compatibility across different connectors
- Data type handling consistency  
- Case sensitivity rules
- Column ordering or naming conventions
- Error message meanings without research

## Communication Protocol

### When Reporting Issues:
1. **Include exact error messages** with error codes
2. **Reference specific documentation** used for troubleshooting
3. **Describe attempted solutions** and their outcomes  
4. **Identify production impact** and urgency level

### When Uncertain:
1. **Document the specific uncertainty**
2. **List what documentation was checked**  
3. **Propose investigation approach**
4. **Recommend validation steps**

---

**Remember: It's better to delay delivery while ensuring production readiness than to deploy solutions that fail in production.**

**Every syntax error, type mismatch, or compatibility issue in production could have been prevented with proper documentation research.**