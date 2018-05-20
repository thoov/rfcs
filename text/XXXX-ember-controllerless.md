- Start Date: 2018-05-13
- RFC PR: (leave this empty)
- Ember Issue: (leave this empty)

# Ember Controllerless

## Summary

> One paragraph explanation of the feature.

Deprecate and remove Controllers. Change the context of route defined templates to be the output from the routes model hook. Move query params to their own service.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected
outcome?

The primary rational behind the change is to simplify the api surface of Ember. Controllers existed in a time before more modern primatives such as services and glimmer components and their role can now be completely reduced using those primatives. Beyond the normal gains when simplifing an api surface area (reduced code size, easier to teach, etc.) we can utilize this opportunity to rethink certain aspects of Ember.


## Detailed design

> This is the bulk of the RFC.

> Explain the design in enough detail for somebody
familiar with the framework to understand, and for somebody familiar with the
implementation to implement. This should get into specifics and corner-cases,
and include examples of how the feature is used. Any new terminology should be
defined here.


There are 3 major changes needed to move away from Controllers.

1) Query Params:

Currently query param functionality exists on both Controllers and Routes like so:

```js
import Controller from '@ember/controller';

export default Controller.extend({
  queryParams: ['category'],
  category: null
});
```

```js
import Route from '@ember/routing/route';

export default Route.extend({
  queryParams: {
    category: {
      refreshModel: true
    }
  },

  model(params) {
    // ...
  }
});
```

The above can be rewritten given a *new* query param service:
```js
import Component from '@ember/component';
import { inject as service } from '@ember/service';
import { computed } from '@ember/object';

export default Component.extend({
  queryParams: service(),

  category: computed('queryParams.route-path.category', {
    get() {
      return this.get('queryParams.route-path.category');
    }
  })
});
```

```js
import Route from '@ember/routing/route';
import { inject as service } from '@ember/service';

export default Route.extend({
  queryParams: service(),

  init() {
    this._super(...arguments);

    this.queryParams.onChange('category', () => {
      this.refresh();
    });
  }


  model(params) {
    // ...
  }
});
```

2) Change the context of route templates to the output of the model hook instead of a controller:

Currently when rendering a template for a route the context for that template is set to the controller. This means when referencing a property in a template: `{{firstName}}` we look at the controller for the `firstName` property.

```js
import Route from '@ember/routing/route';

export default Route.extend({
  model() {
    return {
      firstName: 'Travis',
      lastName: 'Hoover'
    }
  }
});
```

```hbs
First Name: {{model.firstName}}
Last Name: {{model.lastName}}

```

```js
import Route from '@ember/routing/route';

export default Route.extend({
  model() {
    return [{
      name: 'Bob'
    }, {
      name: 'Jane'
    }]
  }
});
```

```hbs
{{#each model as |person|}}
  Name: {{person.name}}
{{/each}}
```

3) Remove / modify existing route methods:

Remove:
controllerFor
resetController
setupController

Modify:
render
renderTemplate

Properties:

Remove:
controller
controllerName
queryParams

## How we teach this

> What names and terminology work best for these concepts and why? How is this
idea best presented? As a continuation of existing Ember patterns, or as a
wholly new one?

> Would the acceptance of this proposal mean the Ember guides must be
re-organized or altered? Does it change how Ember is taught to new users
at any level?

> How should this feature be introduced and taught to existing Ember
users?

## Drawbacks

> Why should we *not* do this? Please consider the impact on teaching Ember,
on the integration of this feature with other existing and planned features,
on the impact of the API churn on existing apps, etc.

> There are tradeoffs to choosing any path, please attempt to identify them here.

## Alternatives

> What other designs have been considered? What is the impact of not doing this?

> This section could also include prior art, that is, how other frameworks in the same domain have solved this problem.

## Unresolved questions

> Optional, but suggested for first drafts. What parts of the design are still
TBD?
