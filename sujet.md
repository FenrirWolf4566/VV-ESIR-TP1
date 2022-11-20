# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. Describe the bug. If possible, say whether the bug is local or global and describe the failure that manifested its presence. Explain the repercussions of the bug for clients/consumers and the company or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues points to the issues considered as solved for the Apache Commons Collections project. Among those issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. Explain the bug and the solution. Did the contributors of the project add new tests to ensure that the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a fault-tolerance verification technique. The company has implemented protocols to test their entire system in production by simulating faults such as a server shutdown. During these experiments they evaluate the system's capabilities of delivering content under different conditions. The technique was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. Read the paper and briefly explain what are the concrete experiments they perform, what are the requirements for these experiments, what are the variables they observe and what are the main results they obtained. Is Netflix the only company performing these experiments? Speculate how these experiments could be carried in other organizations in terms of the kind of experiment that could be performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web browsers. The language was born from a joint effort of the major players in the Web. Its creators presented their design decisions and the formal specification in [a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) published in 2018. The goal of the language is to be a low level, safe and portable compilation target for the Web and other embedding environments. The authors say that it is the first industrial strength language designed with formal semantics from the start. This evidences the feasibility of constructive approaches in this area. Read the paper and explain what are the main advantages of having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification of the language using Isabelle. The paper can be consulted here: https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. This mechanized specification complements the first formalization attempt from the paper. According to the author of this second paper, what are the main advantages of the mechanized specification? Did it help improving the original formal specification of the language? What other artifacts were derived from this mechanized specification? How did the author verify the specification? Does this new specification removes the need for testing?

## Answers

1. Un chercheur hongrois a découvert qu'il était possible de contouner le vérouillage des nouveaux appareils Pixel de Google.
Il s'agit d'une erreur de logique dans le fichier KeyguardHostViewController.java qui amanèrait à une élévation de privilège sans aucune demande administrateur et a été répertoriée sous le CVE : CVE-2022-20465.

    Concretement la manipulation consiste à introduire sa propre carte SIM dans le Google Pixel et a rentrer autant code de dévérouillage necessaire pour bloquer la carte SIM afin de devoir rentrer le code PUK. Celui-ci est facile d'accès : soit écrit l'emballage de la carte SIM, soit par l'opérateur téléphonique.
    Le bug permet alors de bypass le verouillage du téléphone lorsque le code PUK est correct.
    Ce bug a été trouver "un peu par hasard" par le chercheur puis reporté auprès de Google.

    La répercussion chez le client est importante car n'importe quel appareil n'ayant pas fait la mise à jour de patch peut voir ses informations personnelles dérobées lors d'une manipulation physique de son appareil.D'autre part, avec une faille de ce type l'entreprise Google perd en crédibilité et peut être attaquée en justice.

    Ce bug aurait pu etre évititer en testant un scénario : perte du code pin et introduction du code PUK.

    C'est un bug local puisqu'il s'agit de la fonction "dismiss()" qui est utilisée à un mauvais endroit dans le code.

    [Article](https://techcrunch.com/2022/11/14/android-lock-screen-bypass-google-pixel/?guccounter=1&guce_referrer=aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS8&guce_referrer_sig=AQAAALRIRiwcNYizMbOhgCQzsrWt5ESuM2yH912rieftUUZxWFFypXtnbGJOM4EovuOPB9D6GDpNUPm58zDk5uZ1q3Z8Kn9GKeHp1g3PHcGRgdbI1Zw896R1yXXVg4SgZqVI3StAF2QWiqlRIvRMweU6sMkTFrFbdbEA5kWsrZPT56Sp) & [Blog du chercheur](https://bugs.xdavidhu.me/google/2022/11/10/accidental-70k-google-pixel-lock-screen-bypass/)


2. StackOverflowError in SetUniqueList.add() when it receives itself - COLLECTIONS-701

    Le bug est important et concerne les tests.
    Il s'agit d'un bout de code qui fait appel à une méthode "add()" amenant à une fonction infinie.
    ```java
    test() {        
   SetUniqueList l = new SetUniqueList(new LinkedList<Object>()) ;        
   l.add((Object) l) ;    
    }
    ```

    Après une étude approfondie, la recurssion infinie semble venir de `AbstractList.hashCode()` qui invoque `hashCode()` pour chaque élément de la liste.

    D'après changement disponible [ici](https://github.com/apache/commons-collections/commit/1979a6e31067a18c9ede59ad4518f738512eba82), il semble que ce soit un porblème d'ordre de la récursion de `add()` :

    ```java
        public void add(final int index, final E object) {
        // adds element if it is not contained already
        if (set.contains(object) == false) {
            (-) super.add(index, object);
            set.add(object);
            (+) super.add(index, object);
        }
    }
    ```
    En bas du lien github il est indiqué qu'un test `testCollections701()` a été ajouté afin de s'assurer que l'appel de la méthode `add()` ne produise plus une boucle infinie.

3. Le chaos Enginering consiste à introduire continuellement des bugs dans le code afin de voir comment le système se "débrouille" et de trouver la solution à un potentiel problème ne relevant pas forcément du système logiciel.

    **Expériences concrètes :**
    - mettre fin à des instances de machines virtuelles
    - injecter de la latence dans les demandes entre services
    - faire échouer des requêtes entre services
    - faire échouer un service interne.
  
    Le but est de mettre à mal une partie de système et de faire de la redondance, c'est à dire permettre au système d'avoir une alternative.


    **Exigences pour ces expériences :**
    Pour Netflix, l'exigence principale est la disponibilité de l'information : un maximum de client doit pouvoir accès à toutes les fcontionnalités.
    Néanmoins lors de la création de ces perturbation, il est necessaire d'établir un compromis entre le réalisme du test et la compromission du système.


    **Quelles sont les variables qu'ils observent et quels sont les principaux résultats obtenus ? :**
    Les variables les plus observées sont donc la disponibilité (accès aux données), la vitesse (latency) et  la mémoire utilisée.
 
    **Autre entreprises ? Spéculez comment ces expériences pourraient être menées dans d'autres organisations en termes de type d'expérience à réaliser et de variables système à observer pendant les expériences.**
    Toute les entreprises prétendant à une architecture contituée de micro services sont suceptibles d'utiliser le chaos enginering.
    Toute les grosses infrastructures comme celle de Netflix, Google, Facebook ou Amazon l'utilisent notalent dans le soucis de la redondance entre microservices.



4. La spécification formelle permet de définir à l'anvance la motivation, la conception, et la sémantique formelle. Le but est de spécifier le comportement de façon abstraite avant de le mettre osus forme de code.
    
   * Sécurité : elle permet d'éviter les sources non fiable et de protéger les données utilisateurs.
   * Rapidité : optimisation du compilateur  lpermetnnat d'utiliser toute la performance machine.
   * Portable : doit pouvoir toucher toute les plateformes (achitectures, OS, navigateurs...)
   * Compact : le code a transmettre doit être le plus compact possible pour éviter le ralentissement

   **À votre avis, cela signifie-t-il que les implémentations de WebAssembly ne devraient pas être testées ?**
   Le WebAssembly doit d'autant plus etre testé car toute les implémentations ont d'abord été pensées et écrite sur le papier. Certaine incohérence ont pus etre oubliés.


5. **What are the main advantages of the mechanized specification?**
La spécification mécanisé permet la réduction de relation et offre ainsi un langage épuré permettant d'éliminer les comportements indéfinie des spécifications. D'autre part la particularité de cette méthode et que le code est facilement vérifiable.

 **Did it help improving the original formal specification of the language?**
 Au vu de tout les avantages cités précédement, la technique de spécification mécanisé à permi d'améliorer la spécification formelle du langage.

 **What other artifacts were derived from this mechanized specification? How did the author verify the specification?**
 Les "soundness properties" sont des propérités dérivées de la spécification méchanisée. Les vérifications se sont faites via l'ajout de plusieurs Lemmes 

 **Does this new specification removes the need for testing?**
 Il semblerait que le test soit toujours important car de nombreuses erreurs ont été trouvés.