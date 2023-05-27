# Le guide du fronteux$ <small>_(Angular v16 Edition)_</small>

![Observables power!](https://i.imgflip.com/7n8y70.jpg)

## Table des matières

1. [Le pattern déclaratif](#le-pattern-déclaratif)
   1. [Pourquoi ?](#pourquoi)
   2. [Comment faire ?]()
2. [Injection de dépendances]()

<br/>

### Le pattern déclaratif

#### Pourquoi ?

En entreprise nous travaillons sur de gros projets, qui grossissent vite... Et leur compléxité aussi !<br/>

Très rapidement on retrouve des tones de fichiers, avec des tones de lignes de code où l'on (ré)assigne des variables à des tones d'endroits différents... Et bien évidémment dans des `if`, dans des `if`, dans d'autres `if`...<br/>

Prenons un exemple concret que l'on peut rencontrer assez fréquemment, codé en impartif :

```txt
Fonctionnalités :
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
          console.log(searchTerm);
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

Dans le code ci-dessus, on met met à jour **manuellement** la liste des utilisateurs, ainsi que le loading de la table.

Alors qu'il est très simple et de toute petite taille, on peut constater qu'il est déjà assez difficile à lire : les informations de qui se met à jour et quand ne sautent pas aux yeux !

Cela devient alors plus difficile de voir si l'on a oublié de mettre à jour une valeur... Mais d'ailleurs, ne manque-t-il pas quelque chose...?

=> **La problématique** : Quand on écrit ce code, et qu'on est en plein dedans, ça passe encore ! Mais quand il faut remettre les mains dedans quelques jours, semaines, mois ou pire encore, quelques années plus tard, les ennuis commencent (difficulté à retrouvé qui est modifié **où**, **quand**, **par qui** et **pourquoi** ?) !

![Quand tu tentes de déboguer une vieille feature totalement codée en impératif](https://media.giphy.com/media/QMHoU66sBXqqLqYvGO/giphy.gif)

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
