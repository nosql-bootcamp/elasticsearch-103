# ElasticSearch 103

![elastic-logo](https://static-www.elastic.co/fr/assets/blteb1c97719574938d/logo-elastic-elasticsearch-lt.svg?q=700)

**ElasticSearch 103** est un workshope permettant d'expérimenter le scaling sur
ElasticSearch.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>

<span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">elasticsearch-102</span>
par
<a xmlns:cc="http://creativecommons.org/ns#" href="https://github.com/nosql-bootcamp/elasticsearch-101" property="cc:attributionName" rel="cc:attributionURL">Benjamin
CAVY, Sébastien PRUNIER et Chris WOODROW</a> est distribué sous les termes de la licence
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative
Commons - Attribution - NonCommercial - ShareAlike</a>.

## Configuration

Dans le fichier de configuration
`~/progs/elasticsearch-6.0.1/config/elasticsearch.yml`, ajouter la ligne :

```yml
node.max_local_storage_nodes: 2
```

## Lancement du cluster

Lancer deux instances elastic dans deux shell différents à l'aide de la
commande.

```bash
~/progs/elasticsearch-6.0.1/bin/elasticsearch
```

Les deux instances sont déployées par défaut sur les ports `9200` et `9201`.

## Test avec réplication

Les index elasticsearch sont stockés sur des shards (un index est réparti sur
plusieurs shards). Outre les shards "primaires" sur lesquels les données sont
stockés, il existe des shards de réplication, servant uniquement à la
redondances des données. Par défaut elasticsearch configure pour chaque index 5
shards primaires et 5 shards de réplication.

Comptez le nombre d'acteurs en exécutant la commande :

```bash
curl -XGET http://localhost:9200/imdb/actor/_count
```

Coupez une des deux instances (avec CTRL + C dans un des deux shells), puis
exécuter à nouveau cette commande.

On constate qu'il n'y a pas eu de perte de données, car elasticsearch répartit
les shards de manières à ce que chaque instance ait :

* des shards primaires
* les shards répliquant les shards primaires de l'autre instance

## Test sans réplication

Relancer l'instance arrêtée, et exécuter la commande suivante :

```json
PUT imdb/_settings
{
  "number_of_replicas": 0
}
```

Cette commande désactive la réplication pour l'index `imdb` (ne faite pas ça
chez vous !).

Stoppez une des deux instances (comme lors de l'étape précédente), et comptez le
nombre d'acteurs :

```bash
curl -XGET http://localhost:9200/imdb/actor/_count
```

On constate cette fois-ci une perte de données, car les données stockées sur les
shards de l'instance arrêtées ne sont plus répliquées sur l'instance restante.
