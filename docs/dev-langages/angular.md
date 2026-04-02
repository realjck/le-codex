# Angular

Angular est un framework open-source développé par Google, largement utilisé pour construire des applications web dynamiques. Cette fiche mémo vise à fournir une vue d'ensemble d'Angular, depuis l'installation et la configuration jusqu'aux concepts avancés tels que les zones, les animations et la création de bibliothèques de composants réutilisables.

## Installation et Configuration

Angular nécessite d'avoir Node.js et npm (Node Package Manager) d'installés sur votre système.

- Installer Angular CLI globalement sur votre machine en exécutant la commande suivante dans votre terminal : `npm install -g @angular/cli`
- Créer un nouveau projet Angular en exécutant la commande suivante : `ng new my-app`

Un projet Angular typique a la structure suivante :

```
my-app/
|-- e2e/
|-- src/
|   |-- app/
|   |   |-- core/
|   |   |-- features/
|   |   |-- shared/
|   |   |-- ...
|   |-- assets/
|   |-- environments/
|   |-- index.html
|   |-- main.ts
|   |-- styles.css
|-- .angular-cli.json
|-- package.json
|-- tsconfig.json
```

## Les Bases d'Angular

- **Composants** : Un composant est une partie de l'interface utilisateur qui a sa propre logique et son propre template :

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `<h1>{{title}}</h1>`,
  styles: [`h1 { font-weight: normal; }`]
})
export class AppComponent {
  title = 'My First Angular App';
}
```

- **Templates** : Les templates sont utilisés pour définir la structure HTML d'un composant. Ils permettent d'afficher des données dynamiques, de réagir aux événements utilisateur et de contrôler le flux d'exécution de l'application :

```html
<div *ngIf="isLoggedIn">
  <h1>Welcome, {{user.name}}!</h1>
  <button [disabled]="isSaving" (click)="save()">Save</button>
</div>
```

- **Services** : Un service est une classe qui fournit une fonctionnalité spécifique à l'application. Il peut être utilisé pour récupérer des données depuis un serveur, valider des entrées utilisateur ou gérer l'état de l'application :

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  constructor(private http: HttpClient) { }

  getData() {
    return this.http.get('https://api.example.com/data');
  }
}
```

- **Routing** : Le routage permet de naviguer entre les composants en fonction de l'URL. Il permet de créer des applications web monopages (Single Page Applications) en chargeant dynamiquement les composants en fonction de l'URL :

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

## Gestion des États avec NgRx

La gestion d'état consiste à gérer les données de l'application de manière centralisée et prévisible. Imaginez NgRx comme un magasin où vous stockez toutes les données dont votre application a besoin.

- Installer et configurer NgRx dans un projet Angular : `npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools`
- Gérer l'état de l'application à l'aide de stores, d'actions et de réducteurs. Voici un exemple :

```typescript
// actions.ts
import { createAction, props } from '@ngrx/store';

export const loadData = createAction('[Data] Load Data');
export const loadDataSuccess = createAction('[Data] Load Data Success', props<{ data: any }>());
export const loadDataFailure = createAction('[Data] Load Data Failure', props<{ error: any }>());

// reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as DataActions from './actions';

export interface DataState {
  data: any;
  loading: boolean;
}

const initialState: DataState = {
  data: null,
  loading: false
};

export const dataReducer = createReducer(
  initialState,
  on(DataActions.loadData, state => ({ ...state, loading: true })),
  on(DataActions.loadDataSuccess, (state, { data }) => ({ ...state, loading: false, data })),
  on(DataActions.loadDataFailure, state => ({ ...state, loading: false }))
);

// effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { catchError, map, mergeMap } from 'rxjs/operators';
import * as DataActions from './actions';
import { DataService } from './data.service';

@Injectable()
export class DataEffects {
  loadData$ = createEffect(() =>
    this.actions$.pipe(
      ofType(DataActions.loadData),
      mergeMap(() => this.dataService.getData().pipe(
        map(data => DataActions.loadDataSuccess({ data })),
        catchError(error => of(DataActions.loadDataFailure({ error })))
      ))
    )
  );

  constructor(
    private actions$: Actions,
    private dataService: DataService
  ) {}
}
```

## Formulaires

Angular fournit deux approches pour la gestion des formulaires : les formulaires réactifs et les formulaires basés sur les modèles. Les formulaires réactifs permettent de créer des formulaires complexes et personnalisés en utilisant des classes comme `FormControl`, `FormGroup` et `FormArray` :

```typescript
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-reactive-form',
  templateUrl: './reactive-form.component.html',
  styleUrls: ['./reactive-form.component.css']
})
export class ReactiveFormComponent {
  userForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      address: this.fb.group({
        street: [''],
        city: [''],
        zip: ['']
      }),
      hobbies: this.fb.array([
        this.fb.control('')
      ])
    });
  }

  get name() {
    return this.userForm.get('name');
  }

  get email() {
    return this.userForm.get('email');
  }

  get address() {
    return this.userForm.get('address') as FormGroup;
  }

  get hobbies() {
    return this.userForm.get('hobbies') as FormArray;
  }

  addHobby() {
    this.hobbies.push(this.fb.control(''));
  }

  onSubmit() {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    } else {
      console.log('Form is invalid');
    }
  }
}
```

## HttpClient

Pour interagir avec des API externes et échanger des données avec un serveur, Angular met à disposition le service HttpClient. Ce service simplifie la réalisation de requêtes HTTP (GET, POST, PUT, DELETE) et gère les réponses de manière asynchrone à l'aide d'observables. Il permet de configurer les en-têtes, les paramètres de requête et de gérer les erreurs de manière centralisée.

```typescript
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable, throwError } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  private apiUrl = 'https://api.example.com/data';

  constructor(private http: HttpClient) { }

  getData(): Observable<any> {
    return this.http.get(this.apiUrl)
      .pipe(
        retry(3),
        catchError(this.handleError)
      );
  }

  createData(data: any): Observable<any> {
    const headers = new HttpHeaders({ 'Content-Type': 'application/json' });
    return this.http.post(this.apiUrl, data, { headers })
      .pipe(
        catchError(this.handleError)
      );
  }

  updateData(id: number, data: any): Observable<any> {
    const url = `${this.apiUrl}/${id}`;
    const headers = new HttpHeaders({ 'Content-Type': 'application/json' });
    return this.http.put(url, data, { headers })
      .pipe(
        catchError(this.handleError)
      );
  }

  deleteData(id: number): Observable<any> {
    const url = `${this.apiUrl}/${id}`;
    return this.http.delete(url)
      .pipe(
        catchError(this.handleError)
      );
  }

  private handleError(error: any) {
    console.error('An error occurred:', error);
    return throwError(error.message || error);
  }
}
```

## Déploiement et Maintenance

- Générer une version de production de l'application en exécutant la commande suivante : `ng build --configuration production`

## Concepts Avancés

- Les **zones** sont une abstraction qui permet à Angular de suivre les changements dans l'application et de mettre à jour la vue en conséquence. Le changement de détection est le processus par lequel Angular détermine si la vue doit être mise à jour ou non. Le cycle de vie des composants est une série d'événements qui se produisent lors de la création, de la mise à jour et de la destruction d'un composant.

- Utiliser les **animations** pour ajouter des transitions et des effets à l'application. Voici un exemple d'animation simple :

```typescript
import { trigger, state, style, animate, transition } from '@angular/animations';

@Component({
  selector: 'my-component',
  template: `
    <div [@slideInOut]="isVisible ? 'in' : 'out'">
      <!-- content here -->
    </div>
  `,
  animations: [
    trigger('slideInOut', [
      state('in', style({
        transform: 'translateX(0)'
      })),
      state('out', style({
        transform: 'translateX(-100%)'
      })),
      transition('in => out', animate('200ms ease-in-out')),
      transition('out => in', animate('200ms ease-in-out'))
    ])
  ]
})
export class MyComponent {
  isVisible = true;
}
```

- Créer des bibliothèques de composants réutilisables. Vous pouvez créer une bibliothèque de composants réutilisables en utilisant la commande suivante : `ng generate library my-library`

## Conclusion

Pour en savoir plus et approfondir vos connaissances, vous pouvez explorer les ressources en ligne disponibles sur le site officiel d'Angular ([angular.dev](https://angular.dev/)). Vous pourrez en apprendre davantage sur les meilleures pratiques, les nouvelles fonctionnalités et les mises à jour de ce framework.