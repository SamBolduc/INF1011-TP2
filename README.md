# INF1011-TP2

## Patrons de conception

### Adapteur
L'interface entre le système externe de gestion d'inventaire et le programme est faite à l'aide du patron *adapteur*. 
L'interface `InventaireAdapter` définit les méthodes nécessaires pour obtenir un article et mettre à jour l'inventaire après la vente d'un article. 
Il suffit ensuite de fournir une implémentation au programme pour pouvoir interagir avec un système de gestion d'inventaire.

```pseudocode
interface InventaireAdapter{
  Article GetArticleBySKU(string SKU);
  void ReduireInventaire(Article a,int quantite, PointDeVente p)
}

class Inventaire implements InventaireAdapter{
  private ConnInv connection;
  
  Inventaire(/*info necessaires pour se connecter au systeme de gestion d'inventaire*/){
    //effectuer la connection
    connection = //...
  }
  
  override Article GetArticleBySKU(string SKU){
    return new Article(connection.queryArticle(SKU));
  }
  
  override void ReduireInventaire(Article a,int quantite, PointDeVente p){
    connection.queryDecrementInventory(a.SKU, p.id, quantite);
  }
}

class Entreprise{
  //...
  InventaireAdapter inventaire = new Inventaire(/*info necessaires pour se connecter au systeme de gestion d'inventaire*/)
  //...
}
```
