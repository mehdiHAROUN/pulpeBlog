---
title: "101 Angular_netCore"
date: 2021-12-06T22:32:17+01:00
draft: false
---

- install .net 5 https://dotnet.microsoft.com/download/dotnet
- install nodeJs using nvm https://bit.ly/2Hn8HjG to have several version

# chose desired node version

$version = "8.12.0"

# install nvm w/ cmder

choco install cmder
choco install nvm
refreshenv

# install node

nvm ls
nvm use 6.9.1

- make a walking skeleton
  dotnet watch run

npm install --save --legacy-peer-deps (old dependencies)
npm install -g @angular/cli
ng new client
ng serve

to migrate code c# to db :
dotnet ef migrations add UserPasswordAdded
dotnet ef database update

I'm back :

- ng module :
  import
  declaration
  provider
  entrycomponent

index.html => App => Components

\*ngIf

<tr *ngFor='let product of products'>

binding :
c => dom = {{}}
dom => c (click)='toogleImage()'
dom <=> c [(ngModel)]='textInput'

use pipe to transform data
@Pipe({name:'transformName'})
export class convertToSpacePipe implements PipeTransform
{
transform(value:string) : string {
return '';
}
}

.filter return a new array with true condition

arrow function => ...

@Input() rating:number ;
@output() notify: EventEmitter<string> = new EventEmitter<string> ();
<pm-star [rating]=2 (notify)='onNotify($event)'><pm-star>

@Injectable to specify services

root injector vs component injector
@Injectable( { providedIn:'root' })
vs
providers : [ProductService]

constructor(private productService: ProductService)

RxJS :
Observable :
nothing until subsribe
next , error , complete

specify return
http get returns an observable

.pipe(tap(...),catchError(this.handleError))
private handleError (err: httpErrorResponse)

sub! : Subscription ;

Routing :
RouterModule.forRoute(
{ [ path :'products/:id' , component: ProductListComponent] }
{ [ path :'welcome' , component: WelcomeComponent] }  
 { [ path :'' , redirectTo:'welcome'],pathMatch: 'full' }
{ [ path :'**' , component:PageNotFoundComponent] }
)

[routerLink]="[/welcome]"
[routerLink]="['/products/',product.productId]"

<outer-outlet></outer-outlet> to display component selected from routerLink

remove selector from @component if we want to call it from routerLink

private route: ActivatedRoute
ngOnInit() : void {
const in = this.route.snapshot.paramMap.get('id')
}

this.router.navigate(['/producs']);

security (building guard):
ng g g component-name

{ [ path :'products/:id' ,CanActivate : [ProductDetailGuard] component: ProductListComponent] }

Module :

bootstrap array should only be used in the root application module
every component must belong to one and only one angular module

export common module with module if it's used by the shared module
