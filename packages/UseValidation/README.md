### Summary

This package reduces the boilerplate codes need to manage state for form data, modifying and validating it. It provides two intuitive hooks `useFormData` and `useValidation` with clean api. Some of the features it provides are
- Form data can be shared among multiple components without any boilerplate codes.
- Validation state and errors can also be shared among multiple components without any boilerplate codes.
- It usage dotted path of hierarchical form object to modify the value and give flat error object so that is it easer to use with ui events and to show in ui.
- It easily allows you to break form into multiple steps or multiple components. The hook accepts baseDottedPath so that you can only subscribe to part of hierarchical form data and errors.


### Usage

#### Share form and validation data across multiple components
<details>
<summary>Click to expand/collapse this section</summary>
<br/>

##### Create a module for shared form data and validation
```ts
// useTransactionForm.ts
import { CreateSharedFormDataHookWithCleanUp, CreateSharedFormValidationHookWithCleanUp} from "@devnepal/use-form";


export const InitialData = {
    date: null as Date | null,
    amount: 0,
    user: {
        firstName: '',
        lastName: '',
        email: '',
        age: 0,
    }
}

export const {hook: useTransactionForm, cleanUp: cleanUpTransactionForm} = CreateSharedFormDataHookWithCleanUp(InitialData);
export const {hook: useTransactionValidation, cleanUp: cleanUpTransactionValidation} = CreateSharedFormValidationHookWithCleanUp(useTransactionForm);
```

##### Now you can use these hooks in multiple components like below
**Form component One**
```ts
// ExampleTransactionForm.tsx
import React, { ChangeEventHandler, useEffect } from 'react';
import css from '../story.module.css';
import { useTransactionForm, InitialData, useTransactionValidation } from './useTransactionForm';
import * as yup from 'yup';

const TransactionForm = () => {
    const {formData, setFieldValue} = useTransactionForm<typeof InitialData>();
    const { registerValidators, errors } = useTransactionValidation();

    // register validators
    useEffect(() => {
        return registerValidators({
            amount: yup.number().min(1)
        })
    }, [])
    const setFormValue: ChangeEventHandler<HTMLInputElement> = (e) => {
        setFieldValue(e.target.name, e.target.value);
    }

    return (
        <form className={css.form}>
            <label htmlFor="date">Transaction Date</label>
            <input type="date" name="date" id="date" onChange={setFormValue} value={(formData?.date || '')} />

            <label htmlFor="amount">Amount</label>
            <input name="amount" id="amount" onChange={setFormValue} value={(formData?.amount || '')} />
            <div className={css.errorBlock}>
                {
                    Object.entries(errors).map(([key, value]) => (
                        <span><br/>{key}: {value}</span>
                    ))
                }
                <br/>
            </div>
        </form>
    );
}

export default TransactionForm;
```

<br/>

**Form component Two**

```ts
// ExampleUserForm.tsx
import React, { ChangeEventHandler, useEffect } from 'react';
import css from '../story.module.css';
import {useTransactionForm, useTransactionValidation, InitialData } from './useTransactionForm';
import * as yup from 'yup';

const UserForm = () => {
    const {formData: user, setFieldValue} = useTransactionForm<typeof InitialData.user>('user');
    const { registerValidators, errors } = useTransactionValidation('user');

    useEffect(() => {
         return registerValidators({
            firstName: yup.string().required().min(2),
            lastName: yup.string().required().min(2),
            email: yup.string().required().email(),
        })
    }, [])

    const setFormValue: ChangeEventHandler<HTMLInputElement> = (e) => {
        setFieldValue(e.target.name, e.target.value);
    }

    return (
        <form className={css.form}>
            <label htmlFor="firstName">First Name</label>
            <input name="firstName" id="firstName" onChange={setFormValue} value={(user?.firstName || '')} />

            <label htmlFor="lastName">Last Name</label>
            <input name="lastName" id="lastName" onChange={setFormValue} value={(user?.lastName || '')} />

            <label htmlFor="email">Email</label>
            <input name="email" id="email" onChange={setFormValue} value={(user?.email || "")} />
            <div className={css.errorBlock}>
                {
                    Object.entries(errors).map(([key, value]) => (
                        <span><br/>{key}: {value}</span>
                    ))
                }
                <br/>
            </div>
        </form>
    );
}

export default UserForm;

```

>> **Note on `registerValidators`**
>> - It should be called from useEffect to register validators related to current component
>> - It returns a function which can be called to unregister validators. Returning the function from useEffect removes these validators when components are unmounted
>> - validator passed to registerValidator can also be a function that returns error message when data is invalid and empty text otherwise.

##### Simple multi step form

**Root Component**
```ts
// MultiStepForm.tsx
import React, { useState } from 'react';
import UserForm from './ExampleUserForm';
import TransactionForm from './ExampleTransactionForm';
import { useTransactionValidation, cleanUpTransactionForm, cleanUpTransactionValidation } from './useTransactionForm';

const MultiStepForm = ({}) => {
    const [curStep, setCurStep] = useState(0);
    const { isValid } = useTransactionValidation();

    // clean-up when root component is unmounted
     useEffect(() => {
        return () => {
            cleanUpTransactionForm();
            cleanUpTransactionValidation();
        }
    }, []);

    const changeStep = (step: number) => {
        setCurStep((oldStep) => {
            const newStep = oldStep + step;

            if (newStep >= 0 && newStep < 2) {
                return newStep
            }

            return oldStep;
        })
    }

    return (
        <div>
            { curStep === 0 && <UserForm />}
            { curStep === 1 && <TransactionForm />}
            <div>
                <button onClick={() => changeStep(-1)} >Prev</button>
                <button onClick={() => changeStep(1)} disabled={!isValid}>Next</button>
            </div>
        </div>
    )
}

export default MultiStepForm;
```
</details>

#### Scoping hooks specific child of form object
<details>
<summary>Click to expand/collapse this section</summary>
<br/>


```ts
import {useFormData, useValidation} from "./index.ts";
...
// inside your component
const {formData: user, setFieldValue} = useFormData<typeof InitialData.user>(initialData, 'user');
const { registerValidators, errors } = useValidation(user, 'user');
...
```

When hook is scoped with base path (eg. `user` in above example) we can use relative path to access formData or errors eg. we can use `'firstName'` instead of `'user.firstName'`

</details>

#### Using it in single form
<details>
<summary>Click to expand/collapse this section</summary>
<br/>

To use it in only one form you don't need to create a separate module. Instead you can directly use hooks with initial data
```ts
import {useFormData, useValidation} from "./index.ts";
...
// inside your component
const {formData: user, setFieldValue} = useFormData<typeof InitialData.user>(initialData, 'user');
const { registerValidators, errors } = useValidation(user, 'user');

useEffect(() => {
         return registerValidators({
            firstName: yup.string().required().min(2),
            lastName: yup.string().required().min(2),
            email: yup.string().required().email(),
        })
}, []);

...
```
</details>

#### Getting type hints for form data and errors
<details>
<summary>Click to expand/collapse this section</summary>
<br/>

To get type hints on your IDE pass generic type parameters as shown in following example.
```ts
import {useFormData, useValidation} from "./index.ts";
...
// inside your component
const {formData: user, setFieldValue} = useFormData<typeof InitialData.user>(initialData, 'user');
const { registerValidators, errors } = useValidation<typeof validators>(user);
const validators = {
            firstName: yup.string().required().min(2),
            lastName: yup.string().required().min(2),
            email: yup.string().required().email(),
        };
useEffect(() => {
         return registerValidators(validators)
}, []);

...
```
</details>

#### Using function as validators
<details>
<summary>Click to expand/collapse this section</summary>
<br/>

You can use simple function as validator. The function will be called with `flatFieldName, valueToValidate, formData` and it should return error message if the data is invalid otherwise return empty string.

>> **Note Scope hooks with basePath parameter**
>> In previous examples hooks were scoped using basePath parameter eg. `const { registerValidators, errors } = useTransactionValidation('user');`. When basePath is not used we need to use dotted path like below `user.firstName`


```
const { registerValidators, errors } = useTransactionValidation();
useEffect(() => {
         return registerValidators({
                'user.firstName': (fieldName: string, value: string) => (!!value ? '' : 'First name is required'),
                'user.lastName': (fieldName: string, value: string) => (!!value ? '' : 'Last name is required'),
                'user.age': (fieldName: string, age: number) => (age > 0 ? '' : 'Age should be greater than 0'),
                'amount': (fieldName: string, amount: number) => (amount > 0 ? '' : 'Amount should be greater than 0'),
        });
}, []);
```
</details>
