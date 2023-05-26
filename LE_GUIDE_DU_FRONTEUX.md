# Le guide du fronteux$ <small>_(Angular v16 Edition)_</small>

![Observables power!](https://i.imgflip.com/7n8y70.jpg)

## Table des matières

1. [Le pattern déclaratif](#le-pattern-déclaratif)
   1. [Pourquoi ?](#pourquoi)
   2. [Comment ?]()
   3. [Exemples]()
2. [Injection de dépendances]()

<br/>

### Le pattern déclaratif

#### Pourquoi ?

En entreprise nous travaillons sur de gros projets, qui grossissent vite... Et leur compléxité aussi !<br/>

Très rapidement on retrouve des tones de fichiers, avec des tones de lignes de code où l'on (ré)assigne des variables à des tones d'endroits différents... Et bien évidémment dans des `if`, dans des `if`, dans d'autres `if`...<br/>

![L'impératif, ce bordel infini](https://media.giphy.com/media/4JVTF9zR9BicshFAb7/giphy.gif)

Quand on écrit ce code, et qu'on est en plein dedans, ça passe encore ! Mais quand il faut remettre les mains dedans quelques jours, quelques semaines, quelques mois ou pire encore, quelques années plus tard, les ennuis commencent !

![Quand tu tentes de déboguer une vieille feature totalement codée en impératif](https://media.giphy.com/media/QMHoU66sBXqqLqYvGO/giphy.gif)

<br/>

**=> Notre sauveur à tous :** Le pattern déclaratif !
