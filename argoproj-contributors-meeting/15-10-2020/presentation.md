---
marp: true
theme: default
class:
---

# Argo Contributor Experience

* Argo CD UI Development

![bg right contain](https://raw.githubusercontent.com/argoproj/argoproj/master/docs/assets/argo.png)

---

# Stack

* Typescript + SASS
* React
  * ReactRouter
  * ReactForm
* [RxJS](https://github.com/ReactiveX/rxjs)
* [foundation-sites](https://github.com/foundation/foundation-sites)

![bg right contain](https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg)

---

# Directory Structure

**argo-cd/ui/src**
```yaml
app
├── index.tsx         # app entry point
├── applications      # app modules, loaded asynchronous
│   ├── components    # module components implementation
│   └── index.ts      # module entry point
├── help
│   ├── components
│   └── index.tsx
├── login
│   ├── components
│   └── index.tsx
├── settings
│   ├── components
│   └── index.ts
└── shared            # shared components used by all modules
```
---

# State Management

* Plain React ( hooks are preferred )
* DataLoader ( analog of https://github.com/async-library/react-async )
* ReactForm

```html
<DataLoader input={props.match.params.name} load={(name: string) => services.accounts.get(name)}>
{(account: Account) => (
    <div>
      {account.name}
    </div>
)}
</DataLoader>
```

---

# Forms

* `Form` component ([src](https://github.com/tannerlinsley/react-form)) - handles state management and validation
* `FormFied` component ([src](https://github.com/argoproj/argo-ui)) - shows validation errors, handles "touched" state


```html
<Form
    onSubmit={(params: LoginForm) => this.login(params.username, params.password, this.state.returnUrl)}
    validateError={(params: LoginForm) => ({
        username: !params.username && 'Username is required',
        password: !params.password && 'Password is required'
    })}>
    {formApi => (
        <form role='form' onSubmit={formApi.submitForm}>
            <div className='argo-form-row'>
                <FormField formApi={formApi} label='Username' field='username' component={Text} />
            </div>
            ...
        </form>
    )}
</Form>
```

---

# RxJS

Implements Reactive Extensions for JavaScript

```html
<DataLoader
    input={this.props.match.params.name}
    load={name =>
        Observable.combineLatest(                       // produces combined stream of events from:
            this.loadAppInfo(name),                       // * stream of events from Argo CD API
            services.viewPreferences.getPreferences(), q) // * stream of events from browser local storage
          .map(items => {
            ...
          })
        })
    }>
    ...
</DataLoader>
```

---

# SASS + BEM

* BEM - Block Element Modifier http://getbem.com/naming/

```css
.application-conditions {                                # block
    &__condition {                                       # element
        border-left: 5px solid $argo-color-gray-4;

        &--error {                                       # modifier
            border-left-color: $argo-failed-color-dark;
        }
    }
}
```