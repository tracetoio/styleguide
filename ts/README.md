# Traceto.io TypeScript Styleguide
For typescript code in our projects we use [codelyzer](https://github.com/mgechev/codelyzer) as a core lint rules, and `tslint.json` with our configs.
Next you can see our code styleguide based on tslint config + codelyzer.

## Table of Contents
  1. [Types](#types)
  1. [Interfaces](#interfaces)
  1. [Classes & Constructors](#classes--constructors)
  1. [Async Data Processing](#async-data-processing)
  1. [Functions](#functions)
  1. [Imports](#imports)
  1. [Comments](#comments)
  1. [Whitespaces](#whitespaces)
  1. [Comments](#comments)


## Types
  <a name="type--mandatory"></a><a name="1.1"></a>
  - [1.1](#type--mandatory) Types must be explicitly written for all variables, arrays and methods:
    ```ts
    // bad
    const foo = 5;

    bar(a) {
      const includeArray = [1, 2, 3];
      const aIncluded = includeArray.some((element) => element === a);
      const resultObject = {
        array: includeArray,
        digit: a
      };

      return resultObject;
    }

    // good
    interface IBarResult {
      array: Array<number>;
      included: boolean;
    }

    const foo: number = 5;

    bar(a: number): IBarResult {
      const includeArray: Array<number> = [1, 2, 3];
      const aIncluded: boolean = includeArray.some((element: number) => element === a);
      const resultObject: IBarResult = <IBarResult>{
        array: includeArray,
        included: aIncluded
      };

      return resultObject;
    }
    ```
  <a name="type--fullness"></a><a name="1.2"></a>
  - [1.2](#type--fullness) If you do not know or not sure about type of an entity in your code, assign type `any`:
    ```ts
    // bad
    this.anyLibrary
      .getSomething()
      .subscribe(
        () => {},
        (error) => console.log(error)
      );

    bar(a) {
      const includeArray = [1, 2, 3];
      const aIncluded = includeArray.some((element) => element === a);
      const resultObject = {
        array: includeArray,
        digit: a
      };

      return resultObject;
    }

    // good
    this.anyLibrary
      .getSomething()
      .subscribe(
        () => {},
        (error: any) => console.log(error)
      );

    interface IBarResult {
      array: Array<number>;
      included: boolean;
    }

    bar(a: number): IBarResult {
      const includeArray: Array<number> = [1, 2, 3];
      const aIncluded: boolean = includeArray.some((element: number) => element === a);
      const resultObject: IBarResult = <IBarResult>{
        array: includeArray,
        included: aIncluded
      };

      return resultObject;
    }
    ```

## Interfaces
  <a name="interfaces--usecases"></a><a name="2.1"></a>
  - [2.1](#interfaces--usecases) Define interfaces for:
    1. Big and complex data structures.
    1. Interactions between different logic layers.
    1. Also you can define interfaces for some of entities in the system.

  <a name="interfaces--naming"></a><a name="2.2"></a>
  - [2.2](#interfaces--naming) Naming conventions:
    1. For data structures and logic layers interactions add prefix `I` before the name, for example `IVerifier3ReqObj`.
    1. For entities in the system do not add prefix `I`.
  > Why? Interfaces with prefix `I` just describes object for easier and faster developers' understanding what the data is being processed in different methods and layers. In same time interfaces without `I` prefix intuitively looks close to the models from MVP pattern.

## Classes & Constructors
  <a name="classes--declarations-order"></a><a name="3.1"></a>
  - [3.1](#classes--declarations-order) Methods order in classes:
    1. Decorated properties
    1. Private properties
    1. Public properties
    1. Constructor
    1. Framework's methods (for example ngOnInit, if Angular is used)
    1. Private methods
    1. Public methods
    1. Template methods (if frontend)

  <a name="classes--constructor"></a><a name="3.2"></a>
  - [3.2](#classes--constructor) Constructor requirements:
    - All params should be in column under first param and sorted by accessor. First params starts after first opening brackets:
      ```ts
      // bad
      constructor(
        private authService: AuthService,
        protected translateService: TranslateService,
        protected title: Title,
        protected router: Router,
        protected elementRef: ElementRef,
        protected scrollService: ScrollService,
        protected toastr: ToastrService
      ) { }

      // good
      constructor(private authService: AuthService,
                  protected translateService: TranslateService,
                  protected title: Title,
                  protected router: Router,
                  protected elementRef: ElementRef,
                  protected scrollService: ScrollService,
                  protected toastr: ToastrService) { }
      ```
    - Classes have a default constructor if one is not specified.
      ```ts
      // bad
      class someClass {
        foo(): void {
          // any
        }
      }

      // good
      class someClass {

        constructor() { }

        foo(): void {
          // any
        }
      }
      ```
    - Avoid big calculations in contructor. Create functions in class and call them from contructor.

## Async Data Processing
  <a name="async-data--naming"></a><a name="4.1"></a>
  - [4.1](#async-data--naming) Observables should have `$` sign in the end of name as identifier:
    ```ts
    // bad
    class someClass {
      profileChangedObservable: Observable<Profile>;
      ...

      foo(): void {
        ...
        const arrayOfServiceMethodsObservables = service.methods;
        ...
      }
    }

    // good
    class someClass {
      profileChanged$: Observable<Profile>;
      ...

      foo(): void {
        ...
        const arrayOfServiceMethods$ = service.methods;
        ...
      }
    }
    ```

  <a name="async-data--no-global-observable"></a><a name="4.2"></a>
  - [4.2](#async-data--no-global-observable) Avoid using global observable, use High Order Observables instead:
    ```ts
    // bad
    class someClass {
      ...
      foo(): Observably<any> {
        return Observable.create((observer: Observer<any>) => {
          this.observableMethod()
            .subscribe(
              (respone: any) => {
                this.secondObservableMethod(response)
                  .subscribe(
                    () => {
                      observer.complete();
                    }
                  );
              }
            );
        });
      }
      ...
    }

    // good
    class someClass {
      ...
      foo(): Observably<any> {
        return this.observableMethod()
          .pipe(
            concatMap((respone: any) => this.secondObservableMethod(response))
          );
      };
      ...
    }
    ```
  <a name="async-data--wrapping-promises"></a><a name="4.3"></a>
  - [4.3](#async-data--wrapping-promises) Avoid using promises and wrap them into observables and rxjs:
    ```ts
    // bad
    class someClass {
      ...
      foo(): any {
        return this.contractsService
        .getContract(ContractsList.ProfileToken)
        .methods
        .getUserProfileTokenCount()
        .call()
        .then()
        .catch()
      };
      ...
    }

    // good
    class someClass {
      ...
      foo(): Observably<any> {
        return from(this.contractsService
          .getContract(ContractsList.ProfileToken)
          .methods
          .getUserProfileTokenCount()
          .call()
        )
          .pipe(
            catchError((error: any) => throwError(error))
          )
      }
      ...
    }
    ```

## Functions
  <a name="functions--only-arrow-functions"></a><a name="5.1"></a>
  - [5.1](#functions--only-arrow-functions) Use only arrow functions, avoid old plain construction:
    ```ts
    // bad
    class someClass {
      ...
      const filteredArray: Array<any> = someArray.filter(function(item: any) {return item === 1});
      ...
    }

    // good
    class someClass {
      ...
      const filteredArray: Array<any> = someArray.filter((item: any) => item === 1);
      ...
    }
    ```

  <a name="functions--alignments"></a><a name="5.2"></a>
  - [5.2](#functions--alignments) Align method's arguments (when they are over 140 letters) and chains into one column with symmetry:
    ```ts
    // bad
    class someClass {
      ...
      this.service.method()
      .pipe()
      .subscribe();
      ...
    }

    // good
    class someClass {
      ...
      this.service
        .method()
        .pipe()
        .subscribe();
      ...
    }
    ```

## Imports
  <a name="imports--global-imports"></a><a name="6.1"></a>
  - [6.1](#imports--global-imports) Avoid importing the entire library, import only modules you need:
    ```ts
    // bad
    import { MatInputModule } from '@angular/material';
    import { Observable } from 'rxjs/Rx';

    // good
    import { MatInputModule } from '@angular/material/input';
    import { Observable } from 'rxjs';
    ```
  >Check your library's structure. Example above is little contradictory. Anyway as a result you should have only one module imported, not the entire library. 
  <a name="imports--order"></a><a name="6.2"></a>
  - [6.2](#imports--order) Strictly follow next imports order for angular projects with one empty line space between groups:
    1. Angular core modules imports (@angular)
    1. Angular material library imports (@angular/material)
    1. Third party libraries' imports
    1. Store imports
    1. Custom modules
    1. Custom services
    1. Custom components
    1. Custom models
    1. Another custom entities

  > Why? Easier to find import if they are groupped by role.

## Comments
  <a name="comments--methods-declaration"></a><a name="7.1"></a>
  - [7.1](#comments--methods-declaration) Cover with comments in special format methods in controllers:
    ```ts
    // bad
    ...
    private addNewKYCStatus(requestor: IRequestor, requestorPRAddress: string): Observable<KYCStatus> {
      ...
    }
    ...

    // Adds new KYC status to DB
    ...
    private addNewKYCStatus(requestor: IRequestor, requestorPRAddress: string): Observable<KYCStatus> {
      ...
    }
    ...

    // good
    ...
    /**
     * Adds new KYC status to DB
     * @method addNewKYCStatus
     */
    private addNewKYCStatus(requestor: IRequestor, requestorPRAddress: string): Observable<KYCStatus> {
      ...
    }
    ...
    ```

## Whitespaces
  <a name="whitespaces--recommendations"></a><a name="8.1"></a>
  - [8.1](#whitespaces--recommendations) Follow next recommendations for whitespacing:
    - Do not add whitespace between typecasting and variable/object.
    - Do not add whitespace after first curly bracer in object declaration.
    - Align methods' chains and params in symmetry columns
    - Follow tslint rules.
