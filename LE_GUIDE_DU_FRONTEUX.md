# Le guide du fronteux$ <small>_(Angular v16 Edition)_</small>

<!-- ![Observables power!](https://i.imgflip.com/7n8y70.jpg) -->

<br />

## Table des matières

- [Généralités](#généralités)
  - [Le nommage des fichiers](#le-nommage-des-fichiers)
  - [Le découpage du code : **librairie** ou **application** ?](#le-découpage-du-code--librairie-ou-application-)
- [Les templates HTML](#les-templates-html)
  - [Les _inline_ templates](#les-inline-templates)
  - [Le formatting](#le-formatting)
  - [Styliser un composant lui-même](#styliser-un-composant-lui-même)
- [Le pattern déclaratif](#le-pattern-déclaratif)
  - [Pourquoi ?](#pourquoi-)

<br />

## Généralités

### Le nommage des fichiers

Afin de rester cohérent avec la manière de faire d'angular, nous suffixerons tous les fichiers de la manière suivante => `*.<type de fichier>.<ext>`.

En effet lorsque l'on crée des éléments via le CLI d'angular, on retrouve des noms tels que :

- `mon-service.service.ts`
- `mon-component.component.ts`
  - `mon-component.component.html`
  - `mon-component.component.scss`
- `mon-guard.guard.ts`
- etc.

Ainsi pour un nous pourrion avoir quelque chose comme :

- Pour un modèle : `*.model.ts`
- Pour un ou plusieurs types : `*.type.ts`
- Pour un operateur rxjs (custom) : `*.operator.ts`
- Pour des constantes : `*.constant.ts`
- etc.

<br />

### Le découpage du code : **librairie** ou **application** ?

C'est une question que l'on se pose souvent !

> Où suis-je censé mettre le bout de code que je m'apprête à écrire ?

Et la réponse est plutôt simple :

- Le code est _générique_ => on le met dans une librairie

  Par exemple :

  - Un type utilitaire
  - Un opérateur RxJS custom **générique** (qui ne sert pas uniquement à simplifier la lecture d'un flux répondant à un besoin fonctionnel)
  - Un composant (d'**affichage**, tel que les composants material) réutilisable
  - etc.

- Le code n'est _pas (encore) générique_ => on le met dans l'application

  Par exemple :

  - Un type / model répondant au besoin fonctionnel d'une seule application
  - Un opérateur RxJS custom, pour simplifier la lecture d'un flux
  - Un composant fonctionnel, tel qu'une page

> NB : Si du code qui n'était pas générique le devient, on le déplace simplement dans une librairie (et vice versa), tout simplement

<br />
<br />

## Les templates HTML

### Les _inline_ templates

> Doit-on ou peut-on utiliser des inline templates ?

La réponse courte :

- **Doit**-on ? : Non, rien n'y personne ne l'y oblige
- **Peut**-on ? : Oui, rien n'empêche d'en utiliser

> Mais alors quand peut-on utiliser des templates templates ?

La réponse longue :

Il n'y a pas de réponse absolue à cette question mais on peut l'utiliser :

- Si le template contient peu de code (Il peut-être intéressant de l'écrire directement dans le `*.ts` pour avoir toutes les informations au même endroit)
- Si l'on veut utiliser des variables externes, sans passer par la `class` obligatoirement, grâce à l'**interpolation javascript**

  ```ts
  import { UNE_CONSTANTE_EXTERNE } from "path/to/consantes";

  @Component({
    //...
    // Dans le template ci-dessus on ne passe pas par une interpolation
    // Angular {{ valeur }} ou [attribut]="valeur" mai bien par une
    // interpolation javascript ${} d'un template literal (avec ``)
    template: `<p>Ma constante externe : ${UNE_CONSTANTE_EXTERNE}</p>`
  })
  export class MonComponent {}
  ```

- Si l'on en a tout simplement envie (puisque rien ne l'interdit...)!

<br />

### Le formatting

Ici aussi, il n'y aucune règle absolue mais pour garder une cohérence de formatting, d'un développeur à un autre (et pour rendre le code html le plus lisible possible), nous l'écrirons de la sorte :

- pas d'espace entre la **balise ouvrante d'un élément parent** et le **premier élément enfant**
- un seul espace entre **chaque** enfant
- pas d'espace entre la **balise fermante d'un élément parent** et le **dernier élément enfant**

  Exemples de code **incorrect** pour cette règle :

  ```html
  <ul>
    <li>item 1</li>
    <li>item 2</li>
    <li>item 3</li>
  </ul>

  <!-- ou -->
  <ul>
    <li>item 1</li>

    <li>item 2</li>

    <li>item 3</li>
  </ul>

  <!-- etc. -->
  ```

  Exemple de code **correct** pour cette règle :

  ```html
  <ul>
    <li>item 1</li>

    <li>item 2</li>

    <li>item 3</li>
  </ul>
  ```

> NB : Pour le reste **Prettier** s'en chargera !
>
> NB 2 : Avec un inline template, prettier ne pourra pas reforter l'html

<br />

### Styliser un composant lui-même

Bien souvent quand on souhaite rajouter un style css à un composant, directement, on passe par une `div` et on rajoute du style sur cette dernière.

Si l'on n'a **vraiment** besoin de _borner_ le style d'un composant, **c'est souvent la chose à éviter !** Mais pourquoi ?

Prenons un composant très simple (pas forcément le plus pertinent): Un conteneur servant à mettre du texte en valeur, en lui mettant une couleur, par niveau de _sévérité_

```ts
@Component({
  selector: "app-container",
  standalone: true,
  template: `
    <div
      class="container"
      [class.container--info]="severity === 'info'"
      [class.container--warn]="severity === 'warn'"
      [class.container--error]="severity === 'error'"
    >
      <ng-content />
    </div>
  `,
  styles: [
    `
      .container {
        padding: 0.5rem 1rem;
        border-radius: 8px;
      }

      .container--info {
        background: #e9e9ff;
      }

      .container--warn {
        background: #fff2e2;
      }

      .container--error {
        background: #ffe7e6;
      }
    `
  ]
})
export class ContainerComponent {
  @Input() severity: "info" | "warn" | "error" = "info";
}
```

Maintenant imaginons qu'à l'utilisation on ait **besoin** d'une hauteur de 200px :

```ts
@Component({
  selector: "my-app",
  standalone: true,
  imports: [ContainerComponent],
  template: `<app-container style="height: 200px">Ceci est un message de warning !</app-container>`
})
export class App {}
```

Le style inline (utilisé ici uniquement à titre d'exemple, sinon à proscrire !) ci-dessus ne pourra jamais fonctionner car ce style a été mis sur `app-container`. Or si l'on veut que cela fonctionne il faudrait qu'il soit mis sur la `div.container` directement !

> NB : Comme dit au début de cette section, si l'on souhaite justement spécifiquement _borner_ le style d'un composant, alors la manière de faire actuelle et la bonne ! Si l'on souhaite tout de même permette à l'utilisateur d'ajouter quelques règles de style spécifiques à la `div`, on peut alors simplement passer par des `@Input`.

Si l'on souhaite que l'utilisateur de ce composant puisse ajouter des règles de style comme bon lui semble, il faut alors se passer de cette `div` et utiliser l'`app-container` en tant que `div` !

Pour y arriver on peut refactoriser le code de la manière suivante :

```ts
@Component({
  selector: "app-container",
  standalone: true,
  template: "<ng-content />",
  styles: [
    `
      :host {
        display: block;
        padding: 0.5rem 1rem;
        border-radius: 8px;
      }

      :host.info {
        background: #e9e9ff;
      }

      :host.warn {
        background: #fff2e2;
      }

      :host.error {
        background: #ffe7e6;
      }
    `
  ]
})
export class ContainerComponent {
  @HostBinding("class")
  private severityClass: Severity = "info";

  @Input() set severity(severity: Severity) {
    this.severityClass = severity;
  }
}
```

Dans le code ci-dessus, qui fonctionne désormais comme voulu à l'origine, on a :

- Retiré la `div` qui englobait `ng-content`
- Mis l'élément hôte (soit `app-container`) en `display: block` pour que son mode d'affichage soit le même qu'une `div` (pour cela on utilise la `pseudo classe` angular `:host`)
- Remis les classes de sévérité sur `:host` avec un `@HostBinding` pour lui binder directement la classe sélectionnée

<br />
<br />

## Le pattern déclaratif

### Pourquoi ?

En entreprise nous travaillons sur de gros projets, qui grossissent vite... Et leur compléxité aussi !

Très rapidement on retrouve des tones de fichiers, avec des tones de lignes de code où l'on (ré)assigne des variables à des tones d'endroits différents... Et bien évidémment dans des `if`, dans des `if`, dans d'autres `if`...

Prenons un exemple concret que l'on peut rencontrer assez fréquemment, codé en impartif :

```txt
  - Une table répertoriant des utilisateurs par leur
    - nom
    - prénom
    - age
  - Une barre de recherche permettant de filtrer la liste
  - Un bouton d'ajout d'utilisateur
  - Un bouton de suppression d'utilisateur (par ligne)

NB: Pour simplifier le problème on ne mettra pas en place de pagination mais on fera comme si l'on en avait une, avec 10 utilisateurs par page.

NB 2: Toujours pour simplifier, on fera comme si l'on ne reçoit jamais d'erreurs.
```

```ts
interface User {
  id: string;
  lastName: string;
  firstName: string;
  age: number;
}

@Component({
  selector: "user-table",
  standalone: true,
  imports: [
    /* imports */
  ],
  templateUrl: "./user-table.component.html"
})
export class UserTableComponent {
  #userService = inject(UserService);

  users = signal<User[]>([]); // pas de users par défaut

  protected searchInput = new FormControl("", { nonNullable: true });

  protected addUser$$ = new Subject<User>();

  protected removeUser$$ = new Subject<User["id"]>();

  loading = signal(true); // loading par défaut

  constructor() {
    this.#userService.find({ page: 1 }).subscribe((users) => {
      this.users.set(users); // on récupère la liste par défaut des 10 premiers users
      this.loading.set(false); // on retire le loading
    });

    this.searchInput.valueChanges
      .pipe(
        filter(Boolean),
        debounceTime(300),
        switchMap((searchTerm) => {
          this.loading.set(true); // on remet la table en loading
          return this.#userService.search(searchTerm).pipe(
            tap((foundUsers) => {
              this.users.set(foundUsers); // on met à jour la liste de users (limité à 10)
              this.loading.set(false); // on retire de nouveau le loading
            })
          );
        }),
        takeUntilDestroyed()
      )
      .subscribe();

    this.addUser$$
      .pipe(
        exhaustMap((user) => {
          this.loading.set(true);
          return this.#userService.create(user).pipe(
            tap((updatedUsers) => {
              this.users.set(updatedUsers);
              this.loading.set(false);
            })
          );
        }),
        takeUntilDestroyed()
      )
      .subscribe();

    this.removeUser$$
      .pipe(
        exhaustMap((id) => {
          this.loading.set(true);
          return this.#userService.remove(id).pipe(tap((updatedUsers) => this.loading.set(false)));
        }),
        takeUntilDestroyed()
      )
      .subscribe();
  }
}
```

=> [StackBlitz](https://stackblitz.com/edit/stackblitz-starters-3jepdc?file=src%2Fuser-table%2Fuser-table.component.ts)

Dans le code ci-dessus, on met à jour **manuellement** la liste des utilisateurs, ainsi que le loading de la table.

Alors qu'il est (encore) très simple / de toute petite taille, on peut constater qu'il est déjà assez difficile à lire : les informations de qui se met à jour et quand ne sautent pas aux yeux !

Cela devient alors plus difficile de voir si l'on a oublié de mettre à jour une valeur... Mais d'ailleurs, est-ce que tout fonctionne comme attendu...?

=> Quand on écrit ce code, et qu'on est en plein dedans, ça passe encore ! Mais quand il faut remettre les mains dedans quelques jours, semaines, mois ou pire encore, quelques années plus tard, les ennuis commencent (difficulté à retrouvé qui est modifié **où**, \*quand**, **par qui** et **pourquoi\*\* ?) !

<br/>

Refacto du code précédent en mode déclaratif :

```ts
@Component({
  selector: "user-table",
  standalone: true,
  imports: [
    /* imports */
  ],
  templateUrl: "./user-table.component.html"
})
export class UserTableComponent {
  #userService = inject(UserService);

  protected searchInput = new FormControl("", { nonNullable: true });

  findUsers$ = this.#userService.find({ page: 1 });

  searchUsers$ = this.searchInput.valueChanges.pipe(
    filter(Boolean),
    debounceTime(300),
    switchMap((searchTerm) => this.#userService.search(searchTerm))
  );

  @ViewChild("addUser") addUser!: ElementRef;
  #onAddUser$ = fromChildEvent(() => this.addUser, "click").pipe(share());

  addUser$ = this.#onAddUser$.pipe(exhaustMap(() => this.#userService.create(generateMock(UserSchema))));

  protected removeUser$$ = new Subject<User["id"]>();
  removeUser$ = this.removeUser$$.pipe(exhaustMap((id) => this.#userService.remove(id)));

  #users$ = merge(this.findUsers$, this.searchUsers$, this.addUser$, this.removeUser$); // mise à jour des users

  users$ = this.#users$.pipe(startWith([])); // pas de users par défaut

  // affichage du loading pour chaque action
  #showLoading$ = merge(this.searchInput.valueChanges, this.#onAddUser$, this.removeUser$$).pipe(map(() => true));

  loading$ = merge(
    this.#showLoading$,
    this.#users$.pipe(map(() => false)) // suppression du loading à chaque màj users
  ).pipe(startWith(true)); // loading par défaut
}
```

=> [StackBlitz](https://stackblitz.com/edit/stackblitz-starters-8xglgf?file=src%2Fuser-table%2Fuser-table.component.ts)

Alors que l'application est pourtant très simple, on peut facilement se rendre compte du gain en lisibilité que l'on a ici. On sait tout de suite ce qui met à jour les utilisateurs, ainsi que ce qui toggle le loading !

=> On n'est moins sujets à faire des erreurs !
