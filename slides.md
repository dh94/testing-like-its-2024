---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: ./assets/cover.png
# some information about your slides (markdown enabled)
title: Testing Like It's 2024
info: |
  
  Presentation slides for developers.

# apply unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# Testing Like It's 2024

New test patterns & tools to elevate your testing game


<p class="abs-bl bottom-7 left-10">Daniel Jakobsen</p>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes) this is so cool
-->

---

# Daniel Hallel Jakobsen

- **Fullstack Staff Engineer**
- **10+ years** in software development
- **@Vim 5+ Years**
  - Today in Vim OS team
  - Touched all code areas in Vim (backend, frontend, barebone, devx, ci, devops, data, etc) 
- **Speaker** - React Advanced London, NodeTLV meetup, etc.

<img src="/assets/profile.jpeg" class="w-40 h-40 rounded-full top-10 right-30 absolute" />
<img src="/assets/cityLine.svg"  class="w-100vw left-0 bottom-[-100px] absolute" />

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---
transition: fade
---

# The Problem Through Use Case


<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

````md magic-move
```ts {all|8-10}
@injectable()
export class AdapterManager {
  
  constructor(
    /** Injected Dependencies */
  ) {}

  public async loadAdapter(): Promise<void> {
    /** Setup logic for runtime adapter */
  }
}
```
```ts {all|5,10-14,16-18}
@injectable()
export class AdapterManager {
  
  constructor(
    @inject(OsFeatureFlagsClient) private ffClient: OsFeatureFlagsClient,
    /** Other Injected dependencies */
  ) {}

  public async loadAdapter(): Promise<void> {
    const newAdapter = await this.ffClient.getFlag({
      defaultValue: false,
      flagName: 'shouldLoadNewAdapter',
      flagContext: { /** flag context */},
    });

    if (newAdapter) {
      /** Setup logic for V2 adapter */
    } else {
      /** Setup logic for runtime adapter */
    }
  }
}
```
````

---
transition: fade
---

# The Problem Through Use Case


<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  margin-bottom: 0px;
}
</style>

````md magic-move
```ts {all|3,7}
describe('Adapter Manager Tests', () => {
  let container: Container;
  /** Mocks.. */
  
  beforeEach(async () => {
    container = createTestContainer();
    /** Setting up mocks.. */
  });

  it('Should load runtime adapter', async () => {
    /** Setup */
    const adapterManager = container.get<AdapterManager>(AdapterManager);
    await adapterManager.loadAdapter();
    /** Expectations.. */
  });
});
```
```ts {all|4|9-10|14,21}
describe('Adapter Manager Tests', () => {
  let container: Container;
  /** Mocks.. */
  let ffMock: MockProxy<OsFeatureFlagsClient>;
  
  beforeEach(async () => {
    container = createTestContainer();
    /** Setting up mocks.. */
    ffMock = mock();
    container.bind(OsFeatureFlagsClient).toConstantValue(ffMock);
  });
  it('Should load V2 adapter', async () => {
    /** Specific Setup */
    ffMock.mockResolvedValue(true);
    const adapterManager = container.get<AdapterManager>(AdapterManager);
    await adapterManager.loadAdapter();
    /** Expectations.. */
  });
  it('Should load runtime adapter', async () => {
    /** Specific Setup */
    ffMock.mockResolvedValue(false);
    const adapterManager = container.get<AdapterManager>(AdapterManager);
    await adapterManager.loadAdapter();
    /** Expectations.. */
  });
});
```
````

---
transition: slide-left
---

# The Problem Through Use Case

- <span v-mark.red=1>Boilerplate</span> - Setting up the mocks, all the extra imports, etc.
- <span v-mark.red=1>not **DRY**</span> - The same setup code is repeated across tests.

<!-- ####################################################################################################################################### -->

---
transition: slide-left 
---

# How Can You Improve / What are you going to learn?

- **Test Fixtures - What are they?**
- **Solving the previous problem with test fixtures** 
- **What are the advantages compared to before/after hooks**


<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---
layout: quote
transition: slide-up
---
# What are test fixtures?

<h2>It's what creates a <span v-mark.green="1">well known</span> and <span v-mark.green="2">fixed environment</span> in which tests are run so that <span v-mark.green="3">results are repeatable</span>.</h2>

---
transition: slide-up
---

# Test with fixtures

<style>
  .slidev-vclick-current {
    opacity: inherit;
  }
  </style>

```ts {all|16,22|1-14|6,11|6|2|3-4|6|6-9|none|16-21}{maxHeight:'calc(40vh)'}
const ffFixtureIt = it.extend({
  container: async ({}, use) => {
    const container =  createTestContainer();
    return await use(container);
  },
  ffMock: async ({ container }, use) => {
    const ffMock = mock();
    container.bind(OsFeatureFlagsClient).toConstantValue(ffMock);
    return await use(ffMock);
  },
  adapterManager: async ({ contianer }, use) => {
    return await use(container.get<AdapterManager>(AdapterManager));
  }
});
describe('Adapter Manager Tests', () => {
  ffFixtureIt('Should load V2 adapter', async ({ ffMock, adapterManager }) => {
    /** Specific Setup */
    ffMock.mockResolvedValue(true);
    await adapterManager.loadAdapter();
    /** Expectations.. */
  });
  ffFixtureIt('Should load runtime adapter', async ({ ffMock, adapterManager }) => {
    /** Specific Setup */
    ffMock.mockResolvedValue(false);
    const adapterManager = container.get<AdapterManager>(AdapterManager);
    await adapterManager.loadAdapter();
    /** Expectations.. */
  });
});
```
<div class="absolute border top-30 left-40 opacity-0" v-click=9 v-click.hide >
```ts {hide|all|hide}{at:9}
let container: Container;
let adapterManager: AdapterManager;
/** Mocks.. */
let ffMock: MockProxy<OsFeatureFlagsClient>;

beforeEach(async () => {
  container = createTestContainer();
  /** Setting up mocks.. */
  ffMock = mock();
  container.bind(OsFeatureFlagsClient).toConstantValue(ffMock);
  adapterManager = container.get(AdapterManager);
});
```
</div>

---

## Fixtures have a number of advantages over before/after hooks:
<br>

- <span v-mark.green>Encapsulate</span> setup and teardown in the same place so it is easier to write.
- Fixtures are <span v-mark.green>reusable</span> between test files - you can define them once and use in all your tests.
- Fixtures are <span v-mark.green>composable</span> - they can depend on each other to provide complex behaviors.
- Fixtures are <span v-mark.green>on-demand</span> - Only the ones needed by your test will be executed.
- Fixtures simplify <span v-mark.green>grouping</span>. You no longer need to wrap tests in describes that set up environment, and are free to group your tests by their meaning instead.

<br>

<div class="absolute top-80">
```ts {hide|all|hide}{at:2}
import { ffFixtureIt } from '@/__tests__/fixtures';

ffFixtureIt('test with ff', async ({ ffMock }) => {
  // ...
});
```
</div>

<div class="absolute top-80">
```ts {hide|all|hide}{at:3}
const logsFixtureIt = ffFixtureIt.extend({
  logs: async ({ container }, use) => {
    const logMock = mock();
    container.bind(Logger).toConstantValue(logMock);
    return await use(logMock);
  }
});
```
</div>

<div class="absolute top-80">
```ts {hide|all|hide}{at:4}
ffFixtureIt('Should load runtime adapter', async ({ ffMock, adapterManager }) => {
```
</div>





<!-- <a href="https://github.dev/bookmd/bookmd/blob/d89620a206f9d8f6422a9dbb79e7f7f863be97b9/client/vim-os/src/__tests__/inversifyMock.ts" target="_blank" alt="GitHub" title="Open in GitHub" class="text-xl slidev-icon-btn h-10 w-44 opacity-50 !border-none !hover:text-white">
  Vim OS Example
</a> -->

---

## Fixtures use cases
<br>

Fixtures works really well in the following areas:

- Code with dependency injection (NestJS, InversifyJS, etc)
- Complex setup/teardown code that is shared between tests (Database seeding, etc)

<br>

---

## Downside of fixtures
<br>

- Isn't supported by all test frameworks (Jest is dead - long live Vitest).
- Doesn't work well with beforeEach/afterEach hooks in some test frameworks.

<br>

---
layout: center
class: text-center
---

# Questions?

<img src="/assets/vim-globe.svg" alt="Questions?" class="w-50 h-50 " />

<PoweredBySlidev mt-10 />
