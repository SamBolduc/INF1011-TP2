# INF1011-TP2

## Patrons de conception

### Adapteur

L'interface entre le système externe de gestion d'inventaire et le programme est faite à l'aide du patron *adapteur*.
L'interface `InventaireAdapter` définit les méthodes nécessaires pour obtenir un article et mettre à jour l'inventaire
après la vente d'un article.
Il suffit ensuite de fournir une implémentation au programme pour pouvoir interagir avec un système de gestion
d'inventaire.

```Java
interface InventaireAdapter {
    Article GetArticleBySKU(string SKU);

    void ReduireInventaire(Article a, int quantite, PointDeVente p)
}

class Inventaire implements InventaireAdapter {
    private ConnInv connection;

    Inventaire(/*info necessaires pour se connecter au systeme de gestion d'inventaire*/) {
        //effectuer la connection
        connection = //...
    }

    @Override 
    Article GetArticleBySKU(string SKU) {
        Article a = ResultToArticle(connection.queryArticle(SKU));
        return a;
    }

    private Article ResultToArticle(QueryResult r) {
        //Transforme un QueryResult en un Article
    }

    @Override
    void ReduireInventaire(Article a, int quantite, PointDeVente p) {
        connection.queryDecrementInventory(a.SKU, p.id, quantite);
    }
}

class Entreprise {
    //...
    InventaireAdapter inventaire = new Inventaire(/*info necessaires pour se connecter au systeme de gestion d'inventaire*/)
    //...
}
```

### Singleton

Les classes ``RegistreTransaction`` et ``RegistreEmploye`` sont des Singletons. En effet, elles regroupent ensembles la
gestion d'un domaine précis.
Par exemple, le ``RegistreTransaction`` s'occupe de la gestion des ``Transaction`` tandis que le ``RegistreEmploye``
fait la gestion des employées des différents points de ventes.
Pour chaque registre, une seule instance existe dans le système et toute les demandes sont traitées en les appelants.

#### Implémentation de la classe ``RegistreTransaction``

```Java
public class RegistreTransaction {
    private static final RegistreTransaction instance; //Null par défaut, l'instance sera créée au premier appel de la méthode getInstance()

    private RegistreTransaction() {} //Le constructor privé empêche le développeur de créer une nouvelle instance.

    public static RegistreTransaction getInstance() {
        if (instance == null) instance = new RegistreTransaction();
        return instance;
    }

    private List<Transaction> transactions = new ArrayList();

    public Transaction getTransactionById(int idTransaction) {

    }

    public void nouvelleTransaction(AbstractMembre membre) {

    }

    public void annulerTransaction(Transaction transaction) {

    }

    public void supprimerTransaction(Transaction transaction) {

    }

    public void archiverTransaction(Transaction transaction) {

    }

    public void archiverJournee() {

    }

    public void genererRecu(Transaction transaction) {

    }
}
```

#### Implémentation de la classe ``RegistreEmploye``

```Java
public class RegistreEmploye {
    private static final RegistreEmploye instance; //Null par défaut, l'instance sera créée au premier appel de la méthode getInstance()

    private RegistreEmploye() {} //Le constructor privé empêche le développeur de créer une nouvelle instance.

    public static RegistreEmploye getInstance() {
        if (instance == null) instance = new RegistreEmploye();
        return instance;
    }

    private List<Employe> employes = new ArrayList();

    public void ajouterEmploye(String infos, PointDeVente pdv) {

    }

    public void modifierEmploye(Employe employe, String infos) {

    }

    public void supprimerEmploye(Employe employe) {

    }

    public Employe getEmployeById(int idEmploye) {

    }

}
```

### Visitor

Pouvoir générer des rapports dans un système faisant la gestion de transactions dans plusieurs points de ventes est
intéressant.
Comment peut-on faire cela sans devoir implémenter notre logique d'affaire dans chacune des classes de notre système?

Avec le patron visiteur. Celui-ci permet d'implémenter une nouvelle fonctionnalité dans notre système, et ce, de façon
isolé.
Voici une implémentation permettant de collecter des informations concernant les ``Transaction`` et les ``PointDeVente``
pour pouvoir ensuite générer des rapports.

#### Implémentation du visiteur

```Java
public class Transaction implements IVisitable {
    public void accept(IVisitor visitor) {
        visitor.visitTransaction(this);
    }
}

public class PointDeVente implements IVisitable {
    public final RegistreTransaction registreTransaction;

    public void accept(IVisitor visitor) {
        visitor.visitPointDeVente(this);
    }
}

public class RegistreTransaction {
    public List<Transaction> transactions = new ArrayList();
}

public interface IVisitable {
    void accept(IVisitor visitor);
}

public interface IVisitor {
    void visitTransaction(Transaction transaction);

    void visitPointDeVente(PointDeVente pointDeVente);
}

public class ReportVisitor extends IVisitor {

    private int pointDeVenteCount = 0;
    private int transactionCount = 0;
    
    @java.lang.Override
    public void visitPointDeVente(PointDeVente pointDeVente) {
        pointDeVenteCount++;
    }

    @java.lang.Override
    public void visitTransaction(Transaction transaction) {
        transactionCount++;    
    }
}

public class Entreprise {
    private List<PointDeVente> pointDeVente = new ArrayList();

    public void generateReport() {
        ReportVisitor visitor = new ReportVisitor();
        for (PointDeVente pdv : this.pointDeVente) {
            pdv.accept(visitor);
            for (Transaction trans : pdv.registreTransaction.transactions) {
                trans.accept(visitor);
            }
        }
    }
}
```

### Observateur
Le patron de conception *Observateur* est appliqué au projet pour "automatiser" les tâches d'archivage et de génération de reçu lorsqu'une `Transaction` est terminée.

Ainsi, une `Transaction` possède deux observateurs (`ArchivageListener` et `ReçuListener`) qui sont notifiés lorsque `Transaction.ProcéderPaiement()` est terminé (et que le paiement est valide et fait dans l'immédiat). 
Ces deux observateurs déclenchent respectivement les méthodes `RegistreTransaction.ArchiverTransaction(Transaction)`(`ArchivageListener`) et `RegistreTransaction.GénérerReçu(Transaction)`(`ReçuListener`).

#### Implémentation de l'observateur
![observateur](https://user-images.githubusercontent.com/49413363/198856835-fb494b77-6230-445a-96cf-f8a8f116df4d.png)



## Références
https://refactoring.guru/design-patterns/catalog
