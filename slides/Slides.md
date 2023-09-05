---
marp: true
theme: custom
footer: 'Andy Lech - The Secret to Mobile'
--paginate: true
---

<!-- _footer: "" -->

<!-- _class: summary -->

# The Secret to Mobile

- App Architecture
- API Design
- Data Handling

#### Andy Lech

---

<!-- _class: summary -->

### Topics

- Infrastructure differences between Web and Mobile
- Architecturing your Mobile app for better results
- Making better data decisions for your Mobile app
- Replacing exceptions with extensible error models

---

<!-- _class: title -->

## Part 1

* ### Mobile is just like the web but smaller

* ### Right!?

* #### or

* ### Web devs make bad Mobile devs

* ### ... but they can learn

---

### Web vs Mobile - High Level View

<!-- TODO Define API? Add to slides somewhere? -->
<!-- TODO Add pictures of browser/site and device/app -->

<div class="mermaid" style="padding: 20px 0px 10px;">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
flowchart LR
    subgraph Local
        User <--> Browser
    end
    subgraph Remote
        Site <--> DataSource[Data Source]
    end
    Browser <--> Site
</div>

<div class="mermaid" style="padding: 10px 0px 20px;">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
flowchart LR
    subgraph Local
        User <--> App
    end
    subgraph Remote
        API <--> DataSource[Data Source]
    end
    App <--> API
</div>

<div class="commentary" style="padding: 20px 0px 0px;">

* Same thing, right? Problem solved! Crisis averted!  Thank you and good night!

</div>

---

<!-- _class: summary -->

### Web vs Mobile - Comparable (.NET)

- Languages: C#, LINQ, VB.NET, F#, etc.
- Frameworks: .NET, Entity Framework, etc.
- Data: MSSQL, Azure, etc.
- IDEs: Visual Studio, Visual Studio Code, Rider, etc.
- Tools: ReSharper, etc.

</div>

---

<!-- _class: details -->

### Web Sites (Traditional / SPAs)

<div class="columns">

<div>

##### <i>Browser</i>

- Synchronous Browse, Submit (*)
- Caching: Browser (Built-in)
- Resiliency: Browser (Built-in)
- Focus: Interactivity, Data Updates

</div>

<div>

##### <i>Site (Server/Cloud)</i>

- Site Response: Page (HTML/JS)
- State Mgmt: Server, Page
- ORM: Data Source to Server/Cloud
- Caching: CDN, Server/Cloud
- Resiliency: Server/Cloud, Services
- Focus: Layouts, State, Services
- Data Goal: Fat Data Pipes

</div>

</div>

<div class="detail-summary" style="padding: 20px 0px 0px;">

- (*) Single-page application (SPAs) request HTML like synchronous sites and update their content via API calls but still rely on the browser infrastructure

</div>

---

### Web Sites (Traditional / SPAs)

<!-- _class: details -->

<div class="mermaid" style="padding: 20px 0px;">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    [*] --> Page
    state Browser_Stack {
        Page --> Browser
        Browser --> Page : Cache
        Browser --> Page
    }
    Browser --> Web_Controller
    state ServerCloud_Stack {
        Web_Controller --> ORM
        ORM --> Data
        Data --> ORM
        ORM --> Web_Controller
    }
    Web_Controller --> Browser : HTML
</div>

<div class="detail-summary">

- The server/cloud stack is the focus, turning data into pages and layouts
- Complex page layouts are generally built on top of templating frameworks
- SPAs may modify a page layout, but the initial layout comes from the server
- Result: web devs expect fat data pipes in the server/cloud stack so they can get large data payloads (object graphs) all at once and choose what to filter

</div>

---

<!-- _class: details -->

### Mobile Apps

<div class="columns">

<div>

##### <i>App</i>

- Asynchronous REST (GET, POST, etc.)
- State Mgmt: ViewModels, Cache
- ORM: API to App
- Caching: Platform, Local DB (Coded)
- Focus: Interactivity, Data Updates, Layouts, State, API Calls
- Resiliency: API Calls, Network State
- Data Goal: Smart Data Pipes

</div>

<div>

##### <i>API (Server/Cloud)</i>

- API Response: JSON/XML (DTO)
- ORM: Data Source to API
- Focus: Minimum Data, Status Codes
- Resiliency: Server/Cloud, Services
- Data Goal: Smart Data Pipes

</div>

</div>

---

### Mobile Apps

<!-- _class: details -->

<div class="mermaid" style="padding: 10px 0px;">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    [*] --> Page
    state App_Stack {
        direction LR
        Page --> BackEnd
        BackEnd --> Cache : Cache
        Cache: Platform
        Cache: Local DB
        Cache --> BackEnd : Data?
        BackEnd --> Page
    }
    BackEnd --> Api_Controller
    state Api_Stack {
        direction LR
        Api_Controller --> ORM
        ORM --> Data
        Data --> ORM
        ORM --> Api_Controller
    }
    Api_Controller --> BackEnd : JSON
</div>

<div class="detail-summary">

- The app handles interactivity, page layouts, local caching, and calls to the API
- The API stack delivers only data or status codes in response to app requests
- App pages tend to be focused on single tasks so they call the API selectively
- Result: mobile/API devs focus more on reliable, just-in-time data delivery

</div>

---

<!-- _class: details -->

### Web vs Mobile - Different Focuses

<div class="columns">

<div>

##### <i>Web Site</i>

- Synchronous Browse, Submit
- Site Response: Page (HTML/JS)
- State Mgmt: Server, Page
- ORM: Data Source to Server/Cloud
- Client Caching: Browser
- Focus: Heavy Server/Cloud Services
- Resiliency: Server/Cloud
- Data Goal: Fat Data Pipes

</div>

<div>

##### <i>Mobile App</i>

- Asynchronous REST  (GET, POST, etc.)
- API Response: JSON/XML (DTO)
- State Mgmt: App, Cache (OS, DB)
- ORM: Data Source to API, API to App
- Client Caching: App (Custom or OS)
- Focus: Light API Consumption
- Resiliency: API Calls, Server/Cloud
- Data Goal: Smart Data Pipes

</div>

</div>

---

<!-- _class: title -->

## Part 2

* ### Imagine your house was an app ...

---

<!-- _class: details -->

![bg right:55%](./images/isometric-house-profile-concept/17395.jpg)

<div>

##### <i>User Perspective</i>

- Foundation: Has One
- State Management: Room-to-Room, My Stuff is Here
- Service Consumption: Just Works
- Service Production: Somebody Else's Problem
- Resiliency: Generator (Maybe)
- Focus: Location, Amenities, Work, School, Bills, Life

</div>

<div class="attribution" style="padding-top: 20px;">
    <a href="https://www.freepik.com/free-vector/isometric-house-profile-concept_4278104.htm">
    Image by macrovector</a> on Freepik
</div>

---

<!-- _class: details -->

![bg right:40%](./images/plumbing-problems-solution-isometric-infographic-poster/17767.jpg)

<div>

##### <i>Services Perspective (Traditional)</i>

- Foundation: Primary Access
- State Management: Switches, Knobs, Thermostats, Bill Payments, etc.
- Service Consumption: Always
- Service Production: Our Problem
- Resiliency: Better Have a Backup
- Focus: Keeping the Lights On ... and Water, Sewer, etc.

</div>

<div class="attribution" style="padding-top: 20px;">
    <a href="https://www.freepik.com/free-vector/plumbing-problems-solution-isometric-infographic-poster_4283915.htm">
    Image by macrovector</a> on Freepik
</div>

---

<!-- _class: details -->

![bg right:40%](./images/isometric-modern-house/ONTOV20.jpg)

<div>

##### <i>Services Perspective (Solar)</i>

- Foundation: Secondary Access
- State Management: Switches, Knobs, Thermostats, Bill Payments, etc.
- Service Consumption: As Needed
- Service Production: Their Problem
- Resiliency: Battery, Power Grid
- Focus: Providing Electricity when Needed

</div>

<div class="attribution" style="padding-top: 20px;">
    <a href="https://www.freepik.com/free-vector/isometric-modern-house_1086482.htm">
    Image by macrovector</a> on Freepik
</div>

---

<!-- _class: details -->

### Mobile Architecture - Highest Level

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    [*] --> Page
        Page --> BackEnd
        BackEnd --> Cache : Cache
        Cache: Platform
        Cache: Local DB
        Cache --> BackEnd : Data?
        BackEnd --> Page : Data
    BackEnd --> API : Request
    API --> BackEnd : JSON
</div>

---

<!-- _class: details -->

### Mobile Architecture - More Detail

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    [*] --> Page
    Page --> ViewModels : Binding
    ViewModels: ViewModel_1
    ViewModels: ViewModel_2
    ViewModels --> Cache : Cache
    Cache: Platform
    Cache: Local DB
    Cache --> ViewModels : Data?
    ViewModels --> Page : Model
    ViewModels --> API : Request
    API --> ViewModels : JSON
    API: API_1
    API: API_2
</div>

---

<!-- _class: details -->

### Mobile Architecture - Real-world

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    [*] --> Page
    state App_Stack_1 {
        Page --> ViewModels
        ViewModels --> DataService
        ViewModels: ViewModel_1
        ViewModels: ViewModel_2
        DataService --> Cache: Cache
        Cache: Platform
        Cache: Local DB
        Cache --> DataService : Data?
        DataService --> ApiService
        ApiService: ApiService_1
        ApiService: ApiService_2
        ApiService --> DataService : Model
        DataService --> ViewModels : Model
        ViewModels --> Page : Properties
    }
</div>

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
stateDiagram
    direction LR
    state App_Stack_2 {
        direction LR
        ApiTester_1 --> ApiService_1
        ApiService_1 --> ApiTester_1 : Models
        ApiTester_2 --> ApiService_2
        ApiService_2 --> ApiTester_2 : Models
    }
    ApiService_1 --> Api_1
    Api_1 --> ApiService_1
    ApiService_2 --> Api_2
    Api_2 --> ApiService_2
</div>

---

<!-- _class: details -->

### Mobile Architecture - Real-world

<div style="padding: 20px 0px;">

- ViewModels - Moving navigation out of the View allows ViewModels to be tested individually with frameworks like ReactiveUI (RxUI)
- ApiServices - Individually testable to return app Models and not just DTOs
- DataService - Decides whether to call the local cache DB, the platform cache, an API endpoint, or a service based on the requested data and the app lifecycle

</div>

---

<!-- _class: title -->

## Part 3

* ### "It's the data, stupid"

* ### Insert picture of James Carville

* ### Insert picture of The War Room whiteboard

---

<!-- _class: details -->

#### API Design - Code Camp - Schedule

<div class="columns" style="padding: 40px 0px;">

<div>

##### <i>Requirements</i>

- Everybody gets the same schedule
- The schedule changes infrequently
- When the schedule does change, it often involves multiple sessions
- Event Wi-Fi is notoriously fickle

##### <i>Solution</i>

* Send whole schedule w/o redunant data but time-gated and cached

</div>

<div>

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
classDiagram
    Everybody -- Schedule
    Schedule <.. Timeslots
    Schedule <.. Sessions
    Schedule <.. Speakers

</div>

</div>

---

<!-- _class: details -->

#### API Design - Code Camp - Itinerary

<div class="columns" style="padding: 40px 0px;">

<div>

##### <i>Requirements</i>

- Everybody gets the same schedule
- Attendee adds sessions to itinerary
- Attendee shares their itinerary
- Event Wi-Fi is still fickle

##### <i>Solution</i>

* Send itinerary as updated when network is available, managing sync

</div>

<div>

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
classDiagram
    Individual -- Calendar
    Individual -- Schedule
    Calendar <.. Sessions
    Schedule <.. Timeslots
    Schedule <.. Sessions
    Schedule <.. Speakers

</div>

</div>

---

<!-- _class: details -->

#### API Design - Code Camp - Announcements

<div class="columns" style="padding: 30px 0px;">

<div>

##### <i>Requirements</i>

- Everybody gets the same schedule
- Everybody needs to get the same announcements
- Event Wi-Fi is more fickle now that everybody is checking their phones

##### <i>Solution</i>

* Use a highly-available service
* Poll API or wait for real-time data
* Or hope Slack, Twitter, etc. are up

</div>

<div>

<div class="mermaid">
%%{init: {'theme': 'neutral',
    'themeVariables': {'labelBackgroundColor': 'transparent'}}}%%
classDiagram
    Individual -- Announcements
    Individual -- Calendar
    Individual -- Schedule
    Calendar <.. Sessions
    Schedule <.. Timeslots
    Schedule <.. Sessions
    Schedule <.. Speakers

</div>

</div>

---

<!-- _class: details -->

#### API Design + App Design - Other Things

<div class="columns" style="padding: 30px 0px;">

<div>

##### <i>App Design</i>

- Centralized analytics
- Centralized logging
- Auto-generated API wrappers (Refit)
- Centralized API retry (Polly)
- Platform customizations (Xamarin.Essentials)
- Platform dependency-injection

</div>

<div>

##### <i>API Design</i>

- Authorization tokens
- Refresh tokens
- eTags
- Idempotency
- HTTP status codes
- ProblemDetails
- ValidationProblemDetails

</div>

</div>

<div class="detail-summary" style="padding: 20px 0px 0px;">

- And so much more

</div>

---

<!-- _class: title -->

## Part 4

* ### Explain yourself, dammit

* ### Picture of Lewis Black

---

<!-- _class: details -->

<div class="columns" style="padding: 30px 0px;">

<div>

#### ProblemDetails

##### <i>Pros</i>

- Extensible JSON object
- Provides context to status code
- Can report errors or warnings
- Supported in .NET without ASP.NET
- Returned by Refit's ApiException

##### <i>Cons</i>

- Buggy in ASP.NET (still)
- .NET implementations follow JSON

</div>

<div>

```json
# NOTE: '\' line wrapping per RFC 8792
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "An RFC 7807 problem object",
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "format": "uri-reference",
      "description": "A URI reference that identifies the \
problem type."
    },
    "title": {
      "type": "string",
      "description": "A short, human-readable summary of the \
problem type."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status code \
generated by the origin server for this occurrence of the problem.",
      "minimum": 100,
      "maximum": 599
    },
    "detail": {
      "type": "string",
      "description": "A human-readable explanation specific to \
this occurrence of the problem."
    },
    "instance": {
      "type": "string",
      "format": "uri-reference",
      "description": "A URI reference that identifies the \
specific occurrence of the problem. It may or may not yield \
further information if dereferenced."
    }
  }
}
```

</div>

</div>

---

<!-- _class: details -->

### Demo - ProblemReports (PdHelpers)

---

<!-- _class: title -->

## Part 5

### Questions?

<!-- Put this script at the end of Markdown file. -->
<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10.3.1/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true });

window.addEventListener('vscode.markdown.updateContent', function() { mermaid.init() });
</script>
