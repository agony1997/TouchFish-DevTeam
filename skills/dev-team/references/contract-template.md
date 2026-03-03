# CONTRACT Template (LLM-native)

> For TL use in Phase 2. Write as `{date}-CONTRACT.md` to output dir.
> Living document — amendments via TL only, version incremented each change.

---

[SECTION] API Contract
[VERSION] 1

[ENDPOINT] method={METHOD} | path={path} | req={request schema or "none"} | res={response schema} | errors={error codes csv}

[SECTION] Shared Types
[TYPE] name={TypeName} | fields={field:type,field:type,...}

[SECTION] Error Format
[FORMAT] {error response structure, e.g. code:number,message:string,details?:object}

[SECTION] Amendment Log
[AMEND] v={new version} | date={date} | change={what changed} | reason={why}

---

Amendment flow:
1. Worker reports issue → SendMessage TL
2. TL evaluates → modifies CONTRACT.md → increments [VERSION]
3. TL identifies affected Workers → sends shutdown_request
4. TL spawns new Workers with updated CONTRACT
5. QA verifies contract compliance for all related tasks
