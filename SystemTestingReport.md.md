# Panda Lite Api V2 — System Testing Report 

## Project & Team Overview 

This document constitutes the comprehensive, unified Test Report for the **Panda Lite API V2** backend system for the Software Testing & Quality Assurance (STQA) coursework. 

Our quality assurance team divided responsibilities across primary functional domains while cross-testing and verifying edge cases across each other's modules to ensure thorough coverage: 

- **Abu Sayed Rabbi** — Primary: _Authentication, User Registration, Login Security, Restaurant Management & Core Edge Cases_ 

- **Junaed** — Primary: _Order Lifecycle Processing, State Transitions, Role-based Order Permissions_ 

- **Rafi** — Primary: _Menu Item Management, Price/Stock Validations & Availability State Handling_ 

- **Arpon** — Primary: _Rating & Review Logic, API Response Schema Validation_ 

## Section 1: Test Plan 

### 1.1 Overview 

This report documents the system-level black-box testing of the **Panda Lite API V2** REST backend. All endpoint specifications are defined in the project documentation.md . We reference those specifications here and describe our test design decisions, input strategies, and expected behaviors per endpoint group. 

**API Base URL:** As configured in the Postman environment. **Authentication:** Bearer token (JWT) obtained via POST /auth/login . **Roles:** customer , restaurant , rider . 

### 1.2 Test Specifications by Endpoint Group 

- A. Authentication ( POST /auth/register , POST /auth/login ) 

   - **Valid registration** with all required fields (name, email, password, role) — expect 201 . 

   - **Missing individual fields** (name, email, password, role omitted one at a time) — expect 400 . 

- **Email format violations** : malformed emails ( @xyz.com , phone numbers, uppercase duplicates) — expect 400 or 409 . 

- **Password edge cases** : short passwords (< 6 chars), whitespace-only — expect 400 . 

- **Name edge cases** : whitespace-only, excessively long (200+ chars) — expect 400 . 

- **Role validation** : invalid roles, uppercase roles — expect 400 . 

- **Security inputs** : SQL injection payloads — expect no server crash. 

- **Login failures** : wrong password, non-existent email, empty fields, case-sensitive password — expect 401 or 400 . 

B. Restaurant Management ( GET /restaurants , POST /restaurants , PATCH /restaurants/:id ) 

- **GET listing** : valid token, sorting (asc/desc), filtering, no token, invalid params — expect 200 , 400 , 401 . 

- **GET by ID** : valid UUID, non-existent UUID, malformed UUID, no token — expect 200 , 404 , 400 / 404 , 401 . 

- **POST creation** : valid payload, missing fields, whitespace-only fields, duplicate name+address, wrong role — expect 201 , 400 , 403 . 

- **PATCH update** : owner updates own restaurant, cross-owner attempt, whitespace/empty values, invalid data types, no token — expect 200 , 403 , 400 , 401 . 

C. Menu Item Management ( GET /restaurants/:id/menu , POST 

/restaurants/:id/menu , PATCH /menu-items/:id ) 

- **GET menu** : valid restaurant, sorting, filtering, invalid fields, no token, non-existent restaurant — expect 200 , 400 , 401 , 404 . 

- **POST add item** : valid payload, missing fields, negative price, wrong owner, customer role — expect 201 , 400 , 403 , 401 . 

- **PATCH update** : valid update, stock to 0 (expect auto-unavailable), negative stock, wrong owner — expect 200 , 400 , 403 . 

D. Order Processing ( POST /orders , GET /orders , PATCH 

/orders/:id/status , /claim , /cancel ) 

- **POST place order** : valid single/multi-item, missing fields, closed restaurant, zero quantity, non-customer roles — expect 201 , 400 , 403 , 404 . 

- **GET list/single** : customer/restaurant/rider views, sorting, filtering, cross-customer access, unassigned rider access — expect 200 , 403 , 401 . 

- **Timeline & Available** : rider views, sorting, filtering, unauthorized access — expect 200 , 403 , 400 . 

- **Claim** : ready order, non-ready, non-rider roles, already claimed — expect 200 , 400 , 403 , 409 . 

   - **Status transitions** : full lifecycle, illegal jumps, terminal state transitions, wrong role — expect 200 , 400 , 403 . 

   - **Cancel** : placed/accepted, after preparing, non-owner, double cancel — expect 200 , 400 , 403 . 

- E. Rating & Reviews ( POST /orders/:id/rate ) 

   - **Valid rating** : score 1-5 on delivered order — expect 201 . 

   - **Out-of-range** : score 0, 6, 10 — expect 400 . 

   - **Duplicate rating** — expect 409 . Pre-delivery rating — expect 400 . Cross-customer — expect 403 . 

- F. Edge Cases & Security 

   - SQL injection and XSS payloads — expect no 500 crash. 

   - Non-UUID path parameters — expect 400 / 404 , not 500 . 

   - Cross-role boundary enforcement — each role restricted to permitted actions. 

## Section 2: Test Cases 

### 2.1 Authentication — Register ( POST /auth/register ) 

|**ID**|**Short Title**|**Pre-**<br>**conditions**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is a**<br>**Bug**|
|---|---|---|---|---|---|---|---|
|4561d5|Missing<br>Name|None|POST<br>/auth/register<br>with name=""|400|400|Pass|—|
|1b9576|Missing<br>Email|None|POST<br>/auth/register<br>with email=""|400|400|Pass|—|
|39a0da|Missing<br>Email Local<br>Part|None|POST<br>/auth/register<br>with<br>email="@xyz.c<br>om"|400|201|Bug|API accepts<br>malformed<br>email<br>"@xyz.com"<br>and creates<br>account<br>instead of<br>rejecting it|



|**ID**|**Short Title**|**Pre-**<br>**conditions**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is a**<br>**Bug**|
|---|---|---|---|---|---|---|---|
|104e91|Phone<br>Number as<br>Email|None|POST<br>/auth/register<br>with<br>email="015682<br>36240"|400|201|Bug|API accepts a<br>numeric string<br>as valid email<br>and registers<br>user|
|88cd2d|Duplicate<br>Email|User<br>sayed@test<br>.com<br> exists|POST<br>/auth/register<br>with same<br>email|409|409|Pass|—|
|de0000|Missing<br>Password|None|POST<br>/auth/register<br>with<br>password=""|400|400|Pass|—|
|f288a2|Short<br>Password|None|POST<br>/auth/register<br>with<br>password="12<br>34"|400|400|Pass|—|
|1a1124|Missing Role|None|POST<br>/auth/register<br>with role=""|400|400|Pass|—|
|f371a5|SQL<br>Injection in<br>Email|None|POST<br>/auth/register<br>with email="'<br>OR 1=1 --"|400|400|Pass|—|
|39ed24|Uppercase<br>Duplicate<br>Email|User<br>sayed00@t<br>est.com<br>exists|POST<br>/auth/register<br>with<br>"Sayed00@tes<br>t.com<br>"|409|201|Bug|Treats<br>uppercase<br>email as<br>unique,<br>allowing<br>duplicate<br>accounts for<br>same email|
|7b78ed|Uppercase<br>Role|None|POST<br>/auth/register|400|400|Pass|—|



|**ID**|**Short Title**|**Pre-**<br>**conditions**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is a**<br>**Bug**|
|---|---|---|---|---|---|---|---|
||||with<br>role="CUSTO<br>MER"|||||
|0fa9ca|Whitespace-<br>Only<br>Password|None|POST<br>/auth/register<br>with<br>password=" "|400|201|Bug|Accepts<br>whitespace-<br>only<br>password and<br>registers user<br>successfully|
|40e5bc|Whitespace-<br>Only Name|None|POST<br>/auth/register<br>with name=" "|400|201|Bug|Accepts<br>whitespace-<br>only name<br>and creates<br>account|
|51860b|Empty JSON<br>Body|None|POST<br>/auth/register<br>with {}|400|400|Pass|—|
|40057c|Invalid Role|None|POST<br>/auth/register<br>with<br>role="student"|400|400|Pass|—|
|c2b847|Excessive<br>Length<br>Name|None|POST<br>/auth/register<br>with 200+ char<br>name|400|201|Bug|Lacks<br>maximum<br>length<br>validation on<br>string fields;<br>causes<br>database<br>bloat|



2.2 Authentication — Login ( POST /auth/login ) 

|**ID**|**Short Title**|**Pre-**<br>**condition**<br>**s**|**Steps**|**Expec**<br>**ted**|**Actual**|**Verdict**|**Why It Is a Bug**|
|---|---|---|---|---|---|---|---|
|f59f1c|Missing<br>Email|None|POST /auth/login<br>with email=""|400|400|Pass|—|
|c09333|Uppercase<br>Email<br>Login|User<br>sayed@tes<br>t.com<br>exists|POST /auth/login<br>with<br>"SAYED@test.com<br>"|401|401|Pass|—|
|4199f3|Invalid<br>Credentials|None|POST /auth/login<br>with wrong<br>credentials|401|401|Pass|—|
|1e35ac|Empty<br>Password|None|POST /auth/login<br>with password=""|400|400|Pass|—|
|e106f5|SQL<br>Injection in<br>Login|None|POST /auth/login<br>with email='" OR<br>1=1 --'|401|401|Pass|—|
|64f8a7|Wrong<br>Password|User exists|POST /auth/login<br>with wrong<br>password|401|401|Pass|—|
|53d437|Non-<br>existent<br>Email|None|POST /auth/login<br>with unregistered<br>email|401|401|Pass|—|
|a08490|Wrong<br>User Email|Multiple<br>users exist|POST /auth/login<br>with different user's<br>email|401|401|Pass|—|
|190882|Password<br>Case<br>Sensitivity|password=<br>"pass123"|POST /auth/login<br>with "PASS123"|401|401|Pass|—|
|f7a772|Empty<br>Body Login|None|POST /auth/login<br>with {}|400|400|Pass|—|



2.3 Restaurant — GET ( GET /restaurants , GET /restaurants/:id ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|eb6361|List All<br>Restaura<br>nts|Valid<br>token|GET<br>/restaura<br>nts|200|200|Pass|—|
|8f7077|Filter<br>Field<br>Without<br>Value|Valid<br>token|GET<br>/restaura<br>nts?<br>filter_field<br>=name|400|400|Pass|—|
|5158bf|Invalid<br>Filter<br>Field|Valid<br>token|GET<br>/restaura<br>nts?<br>filter_field<br>=bad_col|400|400|Pass|—|
|6b91c8|Filter<br>Value<br>Without<br>Field|Valid<br>token|GET<br>/restaura<br>nts?<br>filter_valu<br>e=Dhanm<br>ondi|400|400|Pass|—|
|4cb1dc|Malforme<br>d UUID<br>Path<br>Param|Valid<br>token|GET<br>/restaura<br>nts/12345<br>-invalid-id|400|500|Bug|API<br>crashes<br>with<br>unhandle<br>d 500<br>exception<br>instead of<br>graceful<br>400/404|
|c2a592|Sort<br>Descendi<br>ng|Valid<br>token|GET<br>/restaura<br>nts?<br>sort_by=<br>desc|200|200|Pass|—|
|2d793d|Filter No|Valid|GET|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||Match|token|/restaura<br>nts?<br>name=xy<br>znotexist|||||
|e32fba|Filter<br>Case<br>Insensitiv<br>e|Valid<br>token|GET<br>/restaura<br>nts?<br>name=BI<br>RYANI|200|200|Pass|—|
|f51b8d|Get by<br>Valid ID|Valid<br>token|GET<br>/restaura<br>nts/:id|200|200|Pass|—|
|1b56cf|Get Non-<br>existent<br>ID|Valid<br>token|GET<br>/restaura<br>nts/:id<br>(user<br>UUID)|404|404|Pass|—|
|c363c7|Filter<br>Partial<br>Name|Valid<br>token|GET<br>/restaura<br>nts?<br>name=bir<br>yani|200|200|Pass|—|
|54f8b4|No Token|None|GET<br>/restaura<br>nts (no<br>auth)|401|401|Pass|—|
|49ef37|Invalid<br>Sort<br>Direction|Valid<br>token|GET<br>/restaura<br>nts?<br>sort_by=i<br>nvalid|400|400|Pass|—|



2.4 Restaurant — POST ( POST /restaurants ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|739d50|Create<br>Restaura<br>nt|Restaura<br>nt role<br>token|POST<br>/restaura<br>nts with<br>valid data|201|201|Pass|—|
|d5a558|Missing<br>Address|Restaura<br>nt token|POST<br>/restaura<br>nts name<br>only|400|400|Pass|—|
|69ec74|Whitespa<br>ce-Only<br>Address|Restaura<br>nt token|POST<br>/restaura<br>nts with<br>address=<br>" "|400|201|Bug|Accepts<br>whitespa<br>ce-only<br>address<br>and<br>creates<br>restauran<br>t|
|aec104|Customer<br>Role<br>Creates|Customer<br>token|POST<br>/restaura<br>nts|403|403|Pass|—|
|c7fee1|Empty<br>Body|Restaura<br>nt token|POST<br>/restaura<br>nts with {}|400|400|Pass|—|
|fe578a|Missing<br>Name|Restaura<br>nt token|POST<br>/restaura<br>nts<br>address<br>only|400|400|Pass|—|
|eaf285|Whitespa<br>ce-Only<br>Name|Restaura<br>nt token|POST<br>/restaura<br>nts with<br>name=" "|400|201|Bug|Accepts<br>whitespa<br>ce-only<br>name<br>and<br>creates<br>restauran|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||||||t|
|8ea6c6|No Token|None|POST<br>/restaura<br>nts (no<br>auth)|401|401|Pass|—|
|ec6af3|Invalid<br>Token|None|POST<br>/restaura<br>nts with<br>random<br>token|401|401|Pass|—|
|5c08bf|Duplicate<br>Restaura<br>nt|Same<br>name+ad<br>dress<br>exists|POST<br>/restaura<br>nts<br>identical<br>data|409|201|Bug|Allows<br>duplicate<br>restauran<br>t creation<br>with<br>identical<br>name<br>and<br>address|
|0919e8|Same<br>Name<br>Diff<br>Address|Same<br>name<br>exists|POST<br>same<br>name, diff<br>address|409|201|Bug|Allows<br>redundan<br>t<br>duplicate<br>restauran<br>t creation|



2.5 Restaurant — PATCH ( PATCH /restaurants/:id ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|||**ns**||||||
|fb6055|Owner<br>Updates|Owner<br>token|PATCH<br>/restaura|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||Name||nts/:id<br>new<br>name|||||
|4ebf4e|Owner<br>Updates<br>isOpen|Owner<br>token|PATCH<br>isOpen=f<br>alse|200|200|Pass|—|
|f14bd5|Cross-<br>Owner<br>Address<br>Update|Diff<br>owner<br>token|PATCH<br>Kabir's<br>restauran<br>t with<br>Sultana's<br>token|403|200|Bug|Cross-<br>owner<br>modificati<br>on<br>allowed;<br>ownershi<br>p<br>authorizat<br>ion<br>bypass|
|0adab1|Whitespa<br>ce<br>Address<br>Update|Owner<br>token|PATCH<br>address=<br>" "|400|200|Bug|Accepts<br>whitespa<br>ce update<br>and<br>overwrite<br>s valid<br>address|
|ff68ef|Non-<br>Boolean<br>isOpen|Owner<br>token|PATCH<br>isOpen="<br>not_a_bo<br>olean"|400|200|Bug|Non-<br>boolean<br>string<br>accepted<br>without<br>schema<br>type<br>validation|
|0e6531|Close<br>Returns<br>Undefine<br>d|Owner<br>token<br>(Arpon)|PATCH<br>isOpen=f<br>alse|200|200|Pass|-|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|ac410a|Reopen<br>Restaura<br>nt|Owner<br>token<br>(Arpon)|PATCH<br>isOpen=tr<br>ue|200|200|Pass|—|
|c86084|Cross-<br>Owner<br>Name<br>Update|Diff<br>owner<br>token|PATCH<br>Kabir's<br>name<br>with<br>Sultana's<br>token|403|200|Bug|Cross-<br>owner<br>restauran<br>t rename<br>permitted<br>without<br>authorizat<br>ion|
|e643bc|Not<br>Found|Owner<br>token|PATCH<br>with<br>menu<br>item<br>UUID|404|404|Pass|—|
|824a60|No Token|None|PATCH<br>(no auth)|401|401|Pass|—|
|3886c3|Empty<br>Name<br>Update|Owner<br>token|PATCH<br>name=""|400|200|Bug|Empty<br>string<br>accepted,<br>wiping<br>restauran<br>t name|



### 2.6 Menu Items — GET ( GET /restaurants/:id/menu ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|13cc50|List Menu<br>Items|Valid<br>token|GET<br>/restaura<br>nts/:id/me|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||nu|||||
|ce5e16|Sort Desc|Valid<br>token|GET ?<br>sort_by=<br>desc|200|200|Pass|—|
|c3125f|Filter by<br>Availabilit<br>y|Valid<br>token|GET ?<br>filter_field<br>=isAvaila<br>ble|200|200|Pass|—|
|ab1bd8|Filter by<br>Price|Valid<br>token|GET ?<br>filter_field<br>=price|200|200|Pass|—|
|8c47a9|No Token|None|GET (no<br>auth)|401|401|Pass|—|
|01232c|Restaura<br>nt Not<br>Found|Valid<br>token|GET<br>(dummy<br>UUID)|404|404|Pass|—|
|0fc858|Invalid<br>Filter<br>Field|Valid<br>token|GET ?<br>filter_field<br>=bad|400|400|Pass|—|



2.7 Menu Items — POST ( POST /restaurants/:id/menu ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|bc04d7|Add<br>Menu<br>Item|Owner<br>token|POST<br>with<br>name,<br>price,<br>stock|201|201|Pass|—|
|e617af|No Token|None|POST|401|401|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||(no auth)|||||
|a9b295|Not<br>Owner|Diff<br>owner<br>token|POST to<br>other's<br>restauran<br>t|403|403|Pass|—|
|0f0431|Customer<br>Role|Customer<br>token|POST<br>menu<br>item|403|403|Pass|—|
|d7af4f|Restaura<br>nt Not<br>Found|Owner<br>token|POST to<br>non-<br>existent<br>restauran<br>t|404|404|Pass|—|
|218fe2|Missing<br>Name<br>and Price|Owner<br>token|POST<br>with<br>stockQua<br>ntity only|400|400|Pass|—|
|ec1db4|Negative<br>Price|Owner<br>token|POST<br>price=-50|400|400|Pass|—|



### 2.8 Menu Items — PATCH ( PATCH /menu-items/:id ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|040bd6|Update<br>Price|Owner<br>token|PATCH<br>price=42<br>0|200|200|Pass|—|
|c4cb82|Zero<br>Stock<br>Availabilit<br>y|Owner<br>token|PATCH<br>stockQua<br>ntity=0|200|200|Bug|isAvail<br>able<br>remains<br>true|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||||||when<br>stock<br>reaches 0<br>instead of<br>flipping to<br>false|
|7c3a44|Re-<br>enable<br>Availabilit<br>y|Owner<br>token|PATCH<br>stock=5,<br>isAvailabl<br>e=true|200|200|Pass|—|
|8fb8fb|No Token|None|PATCH<br>(no auth)|401|401|Pass|—|
|b2770f|Item Not<br>Found|Owner<br>token|PATCH<br>non-<br>existent<br>UUID|404|404|Pass|—|
|cd882f|Not<br>Owner|Diff<br>owner<br>token|PATCH<br>other's<br>item|403|403|Pass|—|
|ce3583|Negative<br>Stock|Owner<br>token|PATCH<br>stockQua<br>ntity=-5|400|500|Bug|Unhandle<br>d server<br>crash<br>with 500<br>error<br>instead of<br>returning<br>400<br>validation|



### 2.9 Orders — Place Order ( POST /orders ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|3c6e1d|Place<br>Single<br>Item<br>Order|Customer<br>token|POST<br>/orders<br>with 1<br>item|201|201|Pass|—|
|bf0dc5|Multi-item<br>Order|Customer<br>token|POST<br>/orders<br>with 2<br>items|201|201|Pass|—|
|defdea|No Token|None|POST<br>/orders<br>(no auth)|401|401|Pass|—|
|47d92b|Restaura<br>nt Owner<br>Places<br>Order|Restaura<br>nt token|POST<br>/orders|403|201|Bug|Restaura<br>nt role<br>bypasses<br>role<br>check<br>and<br>creates<br>order|
|5cff3b|Rider<br>Places<br>Order|Rider<br>token|POST<br>/orders|403|403|Pass|—|
|853b35|Missing<br>restauran<br>tId|Customer<br>token|POST<br>without<br>restauran<br>tId|400|400|Pass|—|
|35c5d2|Missing<br>Items|Customer<br>token|POST<br>without<br>items<br>field|400|400|Pass|—|
|531cd7|Empty<br>Items<br>Array|Customer<br>token|POST<br>items=[]|400|400|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|f0fe71|Restaura<br>nt Not<br>Found|Customer<br>token|POST<br>with fake<br>restauran<br>tId|404|404|Pass|—|
|fa1a43|Cross-<br>Restaura<br>nt Item|Customer<br>token|POST<br>item from<br>diff<br>restauran<br>t|400|400|Pass|—|
|ef6677|Zero<br>Quantity<br>Order|Customer<br>token|POST<br>quantity=<br>0|400|500|Bug|Triggers<br>unhandle<br>d 500<br>crash<br>instead of<br>returning<br>400 Bad<br>Request|
|c9fe4f|Order<br>from<br>Closed<br>Restaura<br>nt|Customer<br>token|POST to<br>closed<br>restauran<br>t|400|400|Pass|—|



### 2.10 Orders — List & View ( GET /orders , GET /orders/:id ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|2d83c0|Schema<br>Check<br>(Arpon)|Customer<br>token|GET<br>/orders|200<br>(array)|200<br>(object)|Bug|API<br>wraps<br>array in{<br>"orders<br>":<br>[...] }|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||||||breaking<br>standard<br>REST<br>schema|
|c9b608|Customer<br>Sees<br>Own<br>Orders|Customer<br>token|GET<br>/orders|200|200|Pass|—|
|5ab393|Restaura<br>nt Sees<br>Orders|Restaura<br>nt token|GET<br>/orders|200|200|Pass|—|
|fe2a2b|Rider<br>Sees<br>Orders|Rider<br>token|GET<br>/orders|200|200|Pass|—|
|14d1d7|Sort<br>Ascendin<br>g|Customer<br>token|GET<br>/orders?<br>sort_by=<br>asc|200|200|Pass|—|
|cf9ceb|Sort<br>Descendi<br>ng|Customer<br>token|GET<br>/orders?<br>sort_by=<br>desc|200|200|Pass|—|
|9ae380|Filter by<br>Placed|Customer<br>token|GET ?<br>status=pl<br>aced|200|200|Pass|—|
|d0f4bf|Filter by<br>Delivered|Customer<br>token|GET ?<br>status=de<br>livered|200|200|Pass|—|
|7508c2|No Token|None|GET<br>/orders<br>(no auth)|401|401|Pass|—|
|17c059|Invalid<br>sort_by|Customer<br>token|GET ?<br>sort_by=r|400|400|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||andom|||||
|2e9654|Customer<br>Views<br>Own<br>Order|Customer<br>token|GET<br>/orders/:i<br>d|200|200|Pass|—|
|1a69ad|Restaura<br>nt Views<br>Order|Restaura<br>nt token|GET<br>/orders/:i<br>d|200|200|Pass|—|
|87540c|Assigned<br>Rider<br>Views<br>Order|Rider<br>token|GET<br>/orders/:i<br>d|200|200|Pass|—|
|d5ff9d|No Token<br>(Single)|None|GET<br>/orders/:i<br>d (no<br>auth)|401|401|Pass|—|
|cb89fd|Order Not<br>Found|Valid<br>token|GET<br>/orders/:i<br>d (fake<br>UUID)|404|404|Pass|—|
|ab70a4|Cross-<br>Customer<br>View|Other<br>customer<br>token|GET<br>/orders/:i<br>d (other's<br>order)|403|200|Bug|Cross-<br>customer<br>access<br>allowed;<br>leaks<br>private<br>customer<br>order<br>data|
|fdb779|Unassign<br>ed Rider<br>View|Unassign<br>ed rider<br>token|GET<br>/orders/:i<br>d|403|200|Bug|Any rider<br>can view<br>unassign<br>ed orders<br>before|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||||||claiming|
|8bfe00|Password<br>Hash<br>Leak<br>(Arpon)|Customer<br>token|GET<br>/orders/:i<br>d (inspect<br>response<br>)|200<br>(sanitized<br>)|200<br>(hash<br>exposed)|Bug|Returns<br>sensitive<br>custome<br>r.passw<br>ordHash<br>($2b$10<br>$...) in<br>response|
|8btime|Timeline<br>Reverse<br>(Arpon)|Customer<br>token|GET<br>/orders/:i<br>d (inspect<br>timeline)|200<br>(chronolo<br>gical)|200<br>(reverse)|Bug|Timeline<br>events<br>returned<br>in reverse<br>order<br>(newest<br>to oldest)|



2.11 Orders — Timeline & Available ( GET /orders/:id/timeline , GET /orders/available ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|a61f30|Customer<br>Views<br>Timeline|Customer<br>token|GET<br>/orders/:i<br>d/timeline|200|200|Pass|—|
|a49300|Filter by<br>actorRole|Customer<br>token|GET ?<br>actorRole<br>=restaura<br>nt|200|200|Pass|—|
|52b8d|Sort Desc<br>Timeline|Customer<br>token|GET ?<br>sort_by=|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||desc|||||
|068fbb|Cancelle<br>d Order<br>Timeline|Customer<br>token|GET<br>timeline<br>(cancelle<br>d order)|200|200|Pass|—|
|5d2b44|No Token<br>Timeline|None|GET<br>timeline<br>(no auth)|401|401|Pass|—|
|06e0da|Order Not<br>Found<br>Timeline|Valid<br>token|GET<br>timeline<br>(fake<br>UUID)|404|404|Pass|—|
|fd73f5|Unauthori<br>zed User<br>Timeline|Other<br>customer|GET<br>timeline<br>(other's<br>order)|403|403|Pass|—|
|28c54b|Rider<br>Views<br>Available|Rider<br>token|GET<br>/orders/a<br>vailable|200|200|Pass|—|
|aadf5b|Filter by<br>deliveryF<br>ee|Rider<br>token|GET ?<br>deliveryF<br>ee=50|200|200|Pass|—|
|da7d6b|No Token<br>Available|None|GET<br>available<br>(no auth)|401|401|Pass|—|
|920447|Customer<br>Cannot<br>View|Customer<br>token|GET<br>/orders/a<br>vailable|403|403|Pass|—|
|9705cb|Restaura<br>nt Cannot<br>View|Restaura<br>nt token|GET<br>/orders/a<br>vailable|403|403|Pass|—|



2.12 Orders — Claim ( PATCH /orders/:id/claim ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|3746c1|Rider<br>Claims<br>Order|Rider<br>token,<br>order<br>ready|PATCH /<br>orders/:id<br>/claim|200|200|Pass|—|
|bc172a|Claim<br>Non-<br>Ready<br>Order|Rider<br>token,<br>order<br>placed|PATCH<br>claim|400|400|Pass|—|
|d9444|Customer<br>Cannot<br>Claim|Customer<br>token|PATCH<br>claim|403|403|Pass|—|
|01d40d|Restaura<br>nt Cannot<br>Claim|Restaura<br>nt token|PATCH<br>claim|403|403|Pass|—|
|62ec9|Order Not<br>Found|Rider<br>token|PATCH<br>claim<br>(fake<br>UUID)|404|404|Pass|—|
|66f65|Already<br>Claimed|Already<br>claimed<br>order|PATCH<br>claim<br>(2nd<br>rider)|409|409|Pass|—|



2.13 Orders — Status Transitions ( PATCH /orders/:id/status ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|4b952c|Accept<br>Order|Restaura<br>nt owner<br>token|PATCH<br>status=ac<br>cepted|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|bd170e|Start<br>Preparing|Restaura<br>nt token|PATCH<br>status=pr<br>eparing|200|200|Pass|—|
|32ac0f|Mark<br>Ready for<br>Pickup|Restaura<br>nt token|PATCH<br>status=re<br>ady_for_<br>pickup|200|200|Pass|—|
|f5cc44|Mark<br>Picked<br>Up|Assigned<br>rider<br>token|PATCH<br>status=pi<br>cked_up|200|200|Pass|—|
|53004b|Mark<br>Delivered|Assigned<br>rider<br>token|PATCH<br>status=de<br>livered|200|200|Pass|—|
|3d5e99|No Token|None|PATCH<br>status (no<br>auth)|401|401|Pass|—|
|28ca85|Missing<br>Status<br>Field|Restaura<br>nt token|PATCH<br>body={}|400|400|Pass|—|
|91d244|Order Not<br>Found|Valid<br>token|PATCH<br>(fake<br>UUID)|404|404|Pass|—|
|3d55cc|Skip Step<br>placed-<br>preparing|Restaura<br>nt token|Skip<br>accepted<br>step|400|400|Pass|—|
|25976b|Invalid<br>Status<br>Value|Valid<br>token|PATCH<br>status="fl<br>ying"|400|400|Pass|—|
|824988|Wrong<br>Restaura<br>nt Owner|Wrong<br>owner<br>token|PATCH<br>status=ac<br>cepted|403|403|Pass|—|
|fd97c|Unassign|Unassign|PATCH|403|200|Bug|Unassign|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||ed Rider<br>Picks Up|ed rider|status=pi<br>cked_up||||ed rider<br>can<br>hijack<br>and mark<br>order as<br>picked up|
|b7f51d|Transition<br>Delivered<br>Order|Delivered<br>order|PATCH<br>status=de<br>livered<br>again|400|200|Bug|Terminal<br>state<br>delivered<br>can be<br>transition<br>ed again|
|0d2477|Re-<br>activate<br>Cancelle<br>d|Cancelle<br>d order|PATCH<br>status=ac<br>cepted|400|200|Bug|Terminal<br>state<br>cancelled<br>order<br>revived<br>back to<br>active|
|ca374e|Illegal<br>Status<br>Jump|Placed<br>order|PATCH<br>status=de<br>livered|400|200|Bug|Skips<br>required<br>lifecycle<br>steps<br>(accept<br>ed,<br>prepari<br>ng,<br>ready)|
|65d9bb|Restaura<br>nt Marks<br>Picked<br>Up|Restaura<br>nt token|PATCH<br>status=pi<br>cked_up|403|200|Bug|Restaura<br>nt role<br>executes<br>rider-only<br>pickup<br>step|
|5b8808|Restaura|Restaura|PATCH|403|200|Bug|Restaura|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||nt Marks<br>Delivered|nt token|status=de<br>livered||||nt role<br>executes<br>rider-only<br>delivery<br>step|
|8bride|Rider ID<br>Reset on<br>Delivery|Delivered<br>order<br>(Arpon)|Inspect<br>riderId<br>field|200<br>(kept)|200 (null)|Bug|Data<br>inconsist<br>ency;<br>delivered<br>order<br>resets<br>riderId<br>tonull|



### 2.14 Orders — Cancel ( PATCH /orders/:id/cancel ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|a10e82|Cancel<br>Placed<br>Order|Customer<br>token,<br>placed|PATCH<br>cancel|200|200|Pass|—|
|a0842|Cancel<br>Delivered<br>Order|Customer<br>token,<br>delivered|PATCH<br>cancel|400|400|Pass|—|
|b8ab57|Cancel<br>No<br>Reason|Customer<br>token,<br>placed|PATCH<br>cancel<br>(no<br>reason)|200|200|Pass|—|
|3a2f7a|Cancel<br>With<br>Reason|Customer<br>token,<br>placed|PATCH<br>cancel<br>reason="<br>Changed|200|200|Pass|—|



|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
||||mind"|||||
|927e2|No Token|None|PATCH<br>cancel<br>(no auth)|401|401|Pass|—|
|78ef5|Order Not<br>Found|Customer<br>token|PATCH<br>cancel<br>(fake<br>UUID)|404|404|Pass|—|
|ff6a2d|Non-<br>owner<br>Customer|Other<br>customer<br>token|PATCH<br>cancel<br>(other's<br>order)|403|403|Pass|—|
|c6ec6e|Restaura<br>nt Cannot<br>Cancel|Restaura<br>nt token|PATCH<br>cancel|403|403|Pass|—|
|7584c2|Rider<br>Cannot<br>Cancel|Rider<br>token|PATCH<br>cancel|403|403|Pass|—|
|70f57d|Cannot<br>Cancel<br>After<br>Preparing|Preparing<br>state|PATCH<br>cancel|400|400|Pass|—|
|a4dc20|Double<br>Cancel|Already<br>cancelled|PATCH<br>cancel<br>again|400|400|Pass|—|



### 2.15 Rating & Reviews ( POST /orders/:id/rate ) 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|39858c|Rating<br>Score<br>Zero|Customer<br>,<br>delivered<br>order|POST<br>rate<br>score=0|400|500|Bug|Validation<br>failure<br>crashes<br>server<br>with 500<br>instead of<br>400|
|bf0205|Rating<br>Score 10|Customer<br>,<br>delivered<br>order|POST<br>rate<br>score=10|400|400|Pass|—|
|bf612a|Rating<br>Score 6<br>(Arpon)|Customer<br>,<br>delivered<br>order|POST<br>rate<br>score=6|400|400|Pass|—|
|a3386c|Duplicate<br>Rating<br>(Arpon)|Already<br>rated<br>order|POST<br>rate<br>again|409|201|Bug|Allows<br>duplicate<br>ratings<br>instead of<br>rejecting<br>with 409<br>Conflict|
|56810a|Rate<br>Undeliver<br>ed Order|Customer<br>, placed<br>order|POST<br>rate|400|400|Pass|—|
|98b7dc|Rate<br>Other<br>Customer<br>Order|Other<br>customer<br>token|POST<br>rate<br>(other's<br>order)|403|403|Pass|—|
|9c60af|Rate<br>Delivered<br>Order|Customer<br>,<br>delivered<br>order|POST<br>rate<br>score=4|201|201|Pass|—|



2.16 Edge Cases & Security 

|**ID**|**Short**<br>**Title**|**Pre-**<br>**conditio**<br>**ns**|**Steps**|**Expecte**<br>**d**|**Actual**|**Verdict**|**Why It Is**<br>**a Bug**|
|---|---|---|---|---|---|---|---|
|9d2447|XSS<br>Payload|Valid<br>token|POST<br>with<br>script tag<br>in name|400/201|201|Pass|—|
|efed6f|SQL<br>Injection<br>in Input|Valid<br>token|POST<br>with<br>DROP<br>TABLE<br>payload|400/201|201|Pass|—|
|f86e87|Customer<br>Updates<br>Status|Customer<br>token|PATCH<br>status=ac<br>cepted|403|403|Pass|—|
|71e631|Rider<br>Places<br>Order|Rider<br>token|POST<br>/orders<br>as rider|403|403|Pass|—|



## Bug Summary Table 

All confirmed defects found across testing, ordered from Authentication to Edge Cases. 

|**#**|**Defect ID**|**Title**|**Endpoint**|**Severity**|**Tester**|
|---|---|---|---|---|---|
|1|39a0da|Missing Email<br>Local Part<br>Accepted|POST<br>/auth/register|Medium|Abu Sayed<br>Rabbi|
|2|104e91|Phone<br>Number<br>Accepted as<br>Email|POST<br>/auth/register|Medium|Abu Sayed<br>Rabbi|
|3|39ed24|Uppercase|POST|High|Abu Sayed|



|**#**|**Defect ID**|**Title**|**Endpoint**|**Severity**|**Tester**|
|---|---|---|---|---|---|
|||Email<br>Creates<br>Duplicate|/auth/register||Rabbi|
|4|0fa9ca|Whitespace-<br>Only<br>Password<br>Accepted|POST<br>/auth/register|High|Abu Sayed<br>Rabbi|
|5|40e5bc|Whitespace-<br>Only Name<br>Accepted|POST<br>/auth/register|Medium|Abu Sayed<br>Rabbi|
|6|c2b847|Unbounded<br>String Length|POST<br>/auth/register|Medium|Abu Sayed<br>Rabbi|
|7|4cb1dc|Malformed<br>UUID Causes<br>500 Crash|GET<br>/restaurants/:i<br>d|High|Abu Sayed<br>Rabbi|
|8|69ec74|Whitespace<br>Address on<br>Create|POST<br>/restaurants|Medium|Abu Sayed<br>Rabbi|
|9|eaf285|Whitespace<br>Name on<br>Create|POST<br>/restaurants|Medium|Abu Sayed<br>Rabbi|
|10|5c08bf|Duplicate<br>Restaurant<br>Allowed|POST<br>/restaurants|Medium|Abu Sayed<br>Rabbi|
|11|0919e8|Same Name<br>Different<br>Address<br>Duplicate|POST<br>/restaurants|Medium|Abu Sayed<br>Rabbi|
|12|f14bd5|Cross-Owner<br>Address<br>Update|PATCH<br>/restaurants/:i<br>d|High|Abu Sayed<br>Rabbi|
|13|c86084|Cross-Owner<br>Name Update|PATCH<br>/restaurants/:i<br>d|High|Abu Sayed<br>Rabbi|



|**#**|**Defect ID**|**Title**|**Endpoint**|**Severity**|**Tester**|
|---|---|---|---|---|---|
|14|0adab1|Whitespace<br>Address on<br>Update|PATCH<br>/restaurants/:i<br>d|Medium|Abu Sayed<br>Rabbi|
|15|3886c3|Empty Name<br>on Update|PATCH<br>/restaurants/:i<br>d|Medium|Abu Sayed<br>Rabbi|
|16|ff68ef|Non-Boolean<br>isOpen<br>Accepted|PATCH<br>/restaurants/:i<br>d|Medium|Abu Sayed<br>Rabbi|
|17|c4cb82|Zero Stock<br>No Auto-<br>Unavailable|PATCH<br>/menu-<br>items/:id|High|Rafi|
|18|ce3583|Negative<br>Stock 500<br>Crash|PATCH<br>/menu-<br>items/:id|High|Rafi|
|19|47d92b|Restaurant<br>Role Places<br>Order|POST /orders|High|Abu Sayed /<br>Junaed /<br>Arpon|
|20|ef6677|Zero Quantity<br>500 Crash|POST /orders|High|Abu Sayed<br>Rabbi|
|21|2d83c0|Orders List<br>Schema<br>Mismatch|GET /orders|Medium|Arpon|
|22|8bfe00|Password<br>Hash Leaked|GET<br>/orders/:id|**Critical**|Abu Sayed /<br>Arpon|
|23|ab70a4|Cross-<br>Customer<br>Order View|GET<br>/orders/:id|High|Junaed|
|24|fdb779|Unassigned<br>Rider Order<br>View|GET<br>/orders/:id|High|Junaed|
|25|8btime|Timeline<br>Reverse|GET<br>/orders/:id|Medium|Arpon|



|**#**|**Defect ID**|**Title**|**Endpoint**|**Severity**|**Tester**|
|---|---|---|---|---|---|
|||Chronology||||
|26|fd97c|Unassigned<br>Rider Hijacks<br>Pickup|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|27|b7f51d|Delivered<br>Order Re-<br>Transitioned|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|28|0d2477|Cancelled<br>Order Re-<br>Activated|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|29|ca374e|Illegal Status<br>Jump|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|30|65d9bb|Restaurant<br>Executes<br>Rider Pickup|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|31|5b8808|Restaurant<br>Executes<br>Rider<br>Delivery|PATCH<br>/orders/:id/sta<br>tus|High|Junaed|
|32|8bride|Delivered<br>Order riderId<br>Reset|PATCH<br>/orders/:id/sta<br>tus|Medium|Arpon|
|33|39858c|Rating Score<br>Zero 500<br>Crash|POST<br>/orders/:id/rat<br>e|High|Arpon|
|34|a3386c|Duplicate<br>Rating<br>Allowed|POST<br>/orders/:id/rat<br>e|Medium|Arpon|



## Section 3: Defect Reports 

Authentication Defects 

#### **DEF-001** — Missing Email Local Part Accepted 

- **Defect ID:** 39a0da 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Send POST /auth/register with email: "@xyz.com" , valid name/password/role 

   - **Expected:** 400 — invalid email format 

- **Actual:** 201 Created — account created with malformed email missing username portion 

   - **Test Case ID:** 39a0da 

#### **DEF-002** — Phone Number Accepted as Email 

- **Defect ID:** 104e91 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Send POST /auth/register with email: "01568236240" 

- **Expected:** 400 — invalid email format 

- **Actual:** 201 Created — account registered with numeric phone number as email 

   - **Test Case ID:** 104e91 

#### **DEF-003** — Uppercase Email Creates Duplicate Account 

- **Defect ID:** 39ed24 

- **Severity:** High 

- **Steps to Reproduce:** 

      1. Register sayed00@test.com 

      2. Send POST /auth/register with email: "Sayed00@test.com" 

- **Expected:** 409 — "Email already exists" 

- **Actual:** 201 Created — uppercase variant treated as separate unique email 

   - **Test Case ID:** 39ed24 

#### **DEF-004** — Whitespace-Only Password Accepted 

- **Defect ID:** 0fa9ca 

- **Severity:** High 

- **Steps to Reproduce:** 

      1. Send POST /auth/register with password: "      " (spaces only) 

- **Expected:** 400 — password cannot be whitespace 

- **Actual:** 201 Created — user registered with unusable password 

   - **Test Case ID:** 0fa9ca 

#### **DEF-005** — Whitespace-Only Name Accepted 

- **Defect ID:** 40e5bc 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Send POST /auth/register with name: "   " (spaces only) 

- **Expected:** 400 — name cannot be whitespace 

- **Actual:** 201 Created — user created with blank display name 

   - **Test Case ID:** 40e5bc 

#### **DEF-006** — Unbounded String Length in Registration 

- **Defect ID:** c2b847 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Send POST /auth/register with 200+ character name 

- **Expected:** 400 — name too long 

- **Actual:** 201 Created — no maximum length validation 

   - **Test Case ID:** c2b847 

Restaurant Defects 

**DEF-007** — Malformed UUID Causes 500 Server Crash 

- **Defect ID:** 4cb1dc 

- **Severity:** High 

- - **Steps to Reproduce:** 

      1. Send GET /restaurants/12345-invalid-id-format 

- **Expected:** 400 or 404 — graceful error 

- **Actual:** 500 Internal Server Error — unhandled exception on non-UUID parameter 

   - **Test Case ID:** 4cb1dc 

#### **DEF-008** — Whitespace-Only Address on Restaurant Create 

- **Defect ID:** 69ec74 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Authenticate as restaurant owner 

      2. Send POST /restaurants with address: "   " 

- **Expected:** 400 — address is missing 

- **Actual:** 201 Created — restaurant created with blank address 

   - **Test Case ID:** 69ec74 

#### **DEF-009** — Whitespace-Only Name on Restaurant Create 

- **Defect ID:** eaf285 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Authenticate as restaurant owner 

      2. Send POST /restaurants with name: "   " 

- **Expected:** 400 — name cannot be whitespace 

- **Actual:** 201 Created — restaurant created with blank name 

   - **Test Case ID:** eaf285 

#### **DEF-010** — Duplicate Restaurant Creation Allowed 

- **Defect ID:** 5c08bf 

- **Severity:** Medium 

- **Steps to Reproduce:** 

   1. Create restaurant with name="Test Cafe", address="123 Main St" 

   2. Send identical POST /restaurants again 

- **Expected:** 409 — duplicate restaurant 

- **Actual:** 201 Created — allows identical duplicates 

- **Test Case ID:** 5c08bf 

**DEF-011** — Same Name Different Address Duplicate Allowed 

- **Defect ID:** 0919e8 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Create restaurant name="Test Cafe", address="123 Main St" 

      2. Send POST /restaurants with name="Test Cafe", address="456 Oak Ave" 

- **Expected:** 409 — duplicate name 

- **Actual:** 201 Created — allows same-name restaurants 

   - **Test Case ID:** 0919e8 

**DEF-012** — Cross-Owner Restaurant Address Update 

- **Defect ID:** f14bd5 

- **Severity:** High 

- **Steps to Reproduce:** 

      1. Authenticate as owner Sultana 

      2. Send PATCH /restaurants/:id targeting Kabir's restaurant with new address 

- **Expected:** 403 — not the owner 

- **Actual:** 200 OK — Sultana modifies Kabir's address 

   - **Test Case ID:** f14bd5 

**DEF-013** — Cross-Owner Restaurant Name Update 

- **Defect ID:** c86084 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Authenticate as owner Sultana 

   2. Send PATCH /restaurants/:id targeting Kabir's restaurant with new name 

**Expected:** 403 — not the owner 

- **Actual:** 200 OK — Sultana renames Kabir's restaurant 

   - **Test Case ID:** c86084 

#### **DEF-014** — Whitespace Address on Restaurant Update 

- **Defect ID:** 0adab1 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Owner sends PATCH /restaurants/:id with address: "   " 

- **Expected:** 400 — address cannot be whitespace 

- **Actual:** 200 OK — overwrites valid address with blank 

   - **Test Case ID:** 0adab1 

#### **DEF-015** — Empty Name on Restaurant Update 

- **Defect ID:** 3886c3 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Owner sends PATCH /restaurants/:id with name: "" 

- **Expected:** 400 — name cannot be empty 

- **Actual:** 200 OK — restaurant name wiped 

   - **Test Case ID:** 3886c3 

#### **DEF-016** — Non-Boolean String Accepted for isOpen 

- **Defect ID:** ff68ef 

- **Severity:** Medium 

- **Steps to Reproduce:** 

   1. Owner sends PATCH /restaurants/:id with isOpen: "not_a_boolean" 

- **Expected:** 400 — type validation error 

- **Actual:** 200 OK — string accepted for boolean field 

- **Test Case ID:** ff68ef 

**DEF-017** — Closing Restaurant Returns isOpen as Undefined 

- **Defect ID:** 0e6531 

- **Severity:** Medium 

- **Steps to Reproduce:** 

   1. Owner sends PATCH /restaurants/:id with isOpen: false 

   2. Inspect response body 

- **Expected:** 200 OK with isOpen: false 

- **Actual:** 200 OK with isOpen: undefined 

- **Test Case ID:** 0e6531 

### Menu Item Defects 

#### **DEF-018** — Zero Stock Does Not Auto-Flip Availability 

- **Defect ID:** c4cb82 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Owner sends PATCH /menu-items/:id with stockQuantity: 0 

- **Expected:** 200 OK with isAvailable: false 

- **Actual:** 200 OK with isAvailable: true — out-of-stock item stays orderable 

- **Test Case ID:** c4cb82 

#### **DEF-019** — Negative Stock Causes 500 Server Crash 

   - **Defect ID:** ce3583 

- **Severity:** High 

- - **Steps to Reproduce:** 

      1. Owner sends PATCH /menu-items/:id with stockQuantity: -5 

   - **Expected:** 400 — must be non-negative 

   - **Actual:** 500 Internal Server Error — unhandled crash 

   - **Test Case ID:** ce3583 

### Order Defects 

#### **DEF-020** — Restaurant Role Can Place Orders 

- **Defect ID:** 47d92b 

- **Severity:** High 

- **Steps to Reproduce:** 

      1. Authenticate with restaurant owner token 

      2. Send POST /orders with valid order data 

- **Expected:** 403 — only customer accounts can place orders 

- **Actual:** 201 Created — restaurant role bypasses restriction 

   - **Test Case ID:** 47d92b 

#### **DEF-021** — Zero Quantity Order Causes 500 Crash 

- **Defect ID:** ef6677 

- **Severity:** High 

- **Steps to Reproduce:** 

      1. Customer sends POST /orders with item quantity: 0 

- **Expected:** 400 — quantity must be at least 1 

- **Actual:** 500 Internal Server Error — unhandled crash 

   - **Test Case ID:** ef6677 

**DEF-022** — Orders List Schema Mismatch 

- **Defect ID:** 2d83c0 

- **Severity:** Medium 

- **Steps to Reproduce:** 

      1. Customer sends GET /orders 

      2. Inspect response body structure 

- **Expected:** 200 OK with top-level array [...] 

- **Actual:** 200 OK with object { "orders": [...] } — violates standard REST contract 

   - **Test Case ID:** 2d83c0 

**DEF-023** — Customer Password Hash Leaked in Order Response 

- **Defect ID:** 8bfe00 

- **Severity: Critical** 

- - **Steps to Reproduce:** 

   1. Customer sends GET /orders/:id 

   2. Inspect the customer object in response 

- **Expected:** Sanitized customer profile (name, email only) 

- **Actual:** Response includes customer.passwordHash ( $2b$10$... ) — exposes bcrypt credentials 

- **Test Case ID:** 8bfe00 

#### **DEF-024** — Cross-Customer Order Data Leak 

- **Defect ID:** ab70a4 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Authenticate as customer Karim 

   2. Send GET /orders/:id using Rahim's order ID 

- **Expected:** 403 — permission denied 

- **Actual:** 200 OK — Karim views Rahim's full order data 

- **Test Case ID:** ab70a4 

#### **DEF-025** — Unassigned Rider Can View Any Order 

- **Defect ID:** fdb779 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Authenticate as rider NOT assigned to the order 

   2. Send GET /orders/:id 

- **Expected:** 403 — permission denied 

- **Actual:** 200 OK — unassigned rider accesses order details 

- **Test Case ID:** fdb779 

#### **DEF-026** — Timeline Returned in Reverse Chronological Order 

- **Defect ID:** 8btime 

- **Severity:** Medium 

- **Steps to Reproduce:** 

   1. Customer sends GET /orders/:id , inspect timeline array 

- **Expected:** Events in chronological order (oldest first) 

- **Actual:** Events in reverse order (newest first) 

- **Test Case ID:** 8btime 

#### **DEF-027** — Unassigned Rider Can Hijack Order Pickup 

- **Defect ID:** fd97c 

- **Severity:** High 

   - **Steps to Reproduce:** 

      1. Authenticate as rider who has NOT claimed the order 

      2. Send PATCH /orders/:id/status with status: "picked_up" 

- **Expected:** 403 — only assigned rider can pick up 

- **Actual:** 200 OK — unassigned rider marks order as picked up 

   - **Test Case ID:** fd97c 

#### **DEF-028** — Delivered Order Can Be Re-Transitioned 

- **Defect ID:** b7f51d 

- **Severity:** High 

   - **Steps to Reproduce:** 

      1. Have a delivered order (terminal state) 

      2. Send PATCH /orders/:id/status with status: "delivered" again 

- **Expected:** 400 — cannot transition from delivered 

- **Actual:** 200 OK — terminal state accepts further transitions 

   - **Test Case ID:** b7f51d 

#### **DEF-029** — Cancelled Order Can Be Re-Activated 

- **Defect ID:** 0d2477 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Have a cancelled order (terminal state) 

   2. Send PATCH /orders/:id/status with status: "accepted" 

- **Expected:** 400 — cannot transition from cancelled 

- **Actual:** 200 OK — cancelled order revived 

- **Test Case ID:** 0d2477 

**DEF-030** — Illegal Status Jump (placed to delivered) 

- **Defect ID:** ca374e 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Have a placed order 

   2. Send PATCH /orders/:id/status with status: "delivered" 

- **Expected:** 400 — invalid state transition 

- **Actual:** 200 OK — skips accepted, preparing, and ready_for_pickup steps 

- **Test Case ID:** ca374e 

**DEF-031** — Restaurant Role Executes Rider-Only Pickup 

- **Defect ID:** 65d9bb 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Authenticate as restaurant owner 

   2. Send PATCH /orders/:id/status with status: "picked_up" 

- **Expected:** 403 — only assigned rider can pick up 

- **Actual:** 200 OK — restaurant performs rider-only transition 

- **Test Case ID:** 65d9bb 

**DEF-032** — Restaurant Role Executes Rider-Only Delivery 

- **Defect ID:** 5b8808 

- **Severity:** High 

- **Steps to Reproduce:** 

   1. Authenticate as restaurant owner 

   2. Send PATCH /orders/:id/status with status: "delivered" 

- **Expected:** 403 — only assigned rider can deliver 

- **Actual:** 200 OK — restaurant marks order delivered 

- **Test Case ID:** 5b8808 

**DEF-033** — Delivered Order Resets riderId to Null 

- **Defect ID:** 8bride 

- **Severity:** Medium 

- **Steps to Reproduce:** 1. Rider delivers order 

   2. Inspect delivered order response 

- **Expected:** riderId retains assigned rider's ID 

- **Actual:** riderId: null — rider assignment wiped on delivery 

- **Test Case ID:** 8bride 

### Rating Defects 

**DEF-034** — Rating Score Zero Causes 500 Crash 

- **Defect ID:** 39858c 

- - **Severity:** High 

- - **Steps to Reproduce:** 

   1. Customer sends POST /orders/:id/rate with score: 0 

- **Expected:** 400 — score must be between 1 and 5 

- **Actual:** 500 Internal Server Error — server crashes on boundary value 

- - **Test Case ID:** 39858c 

#### **DEF-035** — Duplicate Rating Allowed 

- **Defect ID:** a3386c 

- **Severity:** Medium 

- - **Steps to Reproduce:** 

      1. Customer rates delivered order (201 success) 

      2. Customer submits second rating for same order 

- **Expected:** 409 Conflict — already rated 

- **Actual:** 201 Created — duplicate rating accepted 

   - **Test Case ID:** a3386c 

## Section 4: Individual Reflections 

**Abu Sayed Rabbi (011221327)** — I served as QA Lead coordinating our test repository, environment configurations, and final report synthesis, while directly owning the Authentication and Restaurant modules. My testing uncovered 17 defects (SL #1–#16, #21) and co-identified 2 cross-module security flaws (SL #20, #23). Through boundary and input-sanitization analysis, I identified registration flaws accepting malformed emails (SL #1, #2), case-sensitivity collisions (SL #3), whitespace-only inputs (SL #4, #5, #8, #9, #14), and unbounded string lengths (SL #6). In restaurant CRUD operations, I caught an unhandled 500 crash on malformed UUIDs (SL #7) and severe Broken Object Level Authorization (BOLA) vulnerabilities permitting cross-owner modifications (SL #12, #13). Additionally, I engineered negative edge cases across orders— uncovering the zero-quantity HTTP 500 crash (SL #21)—and verified that security injection payloads (SQLi, XSS) did not destabilize the service. 

**Md Junaed Khandakar (0112330072)** First and foremost, I explored the order lifecycle, which is the stateful and permission sensitive feature of the Panda Lite API. I tested access for each role to GET /orders/:id and PATCH /orders/:id/status with regards to data leak, access by an unassigned rider, illegal state transition, and terminal state breach. As a result, I found out that there are quite a few problems in this context, like the ability of a customer to get full order information about another customer (Bug #23), unassigned rider being able to take over the delivery status of orders he is not even assigned to (Bugs #24 and #26), and a number of problems with state machine transition: redelivering delivered order (Bug #27), restoring a cancelled order back into active state (Bug #28), and skipping necessary workflow steps in order to change state to the one needed (Bug #29). Additionally, I discovered two role-boundary violations: restaurant account being capable of performing rider-only status transitions of marking an order as picked up (Bug #30) and as delivered (Bug #31). Moreover, I assisted in writing the rating section, helping to confirm that rating of delivered order works and checking restaurant role order placement bug 

**K M Marziul Karim (Rafi) (0112310027)** For this assignment, my focus was Menu Item Management. I tested GET /restaurants/:id/menu, POST /restaurants/:id/menu, and PATCH /menu-items/:id, covering both happy and failure paths. I built cases for listing and filtering menu items, adding new items with role and ownership checks, and updating price, stock, and availability. Instead of just checking status codes, I went back to the specific rules the documentation laid out, and that's where I found Bugs #17 and #18. Bug #17: updating stockQuantity to 0 is supposed to automatically flip isAvailable to false, but the API leaves it true, so a customer could still order an item that's actually out of stock. Bug #18: sending a negative stockQuantity should return a clean 400 validation error, but instead the server crashes with a 500, meaning the input isn't being validated before it's used. My approach throughout was to test each endpoint against what the docs actually promised, not just whether a request technically worked, since that's exactly where these two bugs were hiding. Junaed is typing 

**MD Shazan Mahmud Arpon (0112310351)** - I led the functional testing of the Ratings & Reviews module along with Schema Integrity and Response Structure Consistency across the API lifecycle. My testing identified 6 primary defects (SL #17, #22, #26, #33, #34, #35) and 

contributed to cross-module validations (SL #19, #20). In the rating module, I uncovered an unhandled HTTP 500 server crash on a score of zero (SL #34) and verified that duplicate ratings on the same order are erroneously permitted (SL #35). While validating response contracts, I caught a schema mismatch on GET /orders returning an object wrapper instead of an array (SL #22), reversed timeline event ordering (SL #26), and data corruption bugs where isOpen returns undefined upon restaurant closure (SL #17) and riderId is wiped to null after order delivery (SL #33). Additionally, I assisted in identifying the HTTP 500 crash on negative stock quantities (SL #19). 

