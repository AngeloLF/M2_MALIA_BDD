# M2 MALIA - Big Data Management - Rendu TP2

Rendu par Angelo Lamure-Fontanini. 

Dans ce TP, le but sera de tester les performances d'un *map reduce*, sur le comptage de mots dans un grand texte (+31 millions de mots). Pour cela, on va effectuer 3 méthodes différentes :

* **Partie 1 : Mono processeur** : On effectue un comptage classique, avec un processeur
* **Partie 2 : Multi processeurs, sans *map reduce*** : On effectue un comptage classique, mais en parallèle sur plusieurs coeurs
* **Partie 3 : Multi processeurs, avec *map reduce*** : On utilise une méthode *map reduce* pour faire le calcul

On a enfin une dernière partie pour présenter les résultats.

## Partie 1 : Mono processeur

Dans cette partie, on va faire un compate classique. Pour cela, on va créer une fonction *loading* qui va être utiliser dans toute les partie pour lire le texte, ainsi que le pré-traiter : enlever les majuscule, les points, les retour à la ligne, etc. On va aussi créer un fonction *countMots*.

```python
def loading(file="large-txt.txt"):

    with open(file, "r") as f:

        data = re.findall(r'\b\w+\b', f.read().lower())

    return data
```

```python
def countMots(data, debug=False):
    """
    Permet de compter les mots dans la liste data
    """

    cnt = dict()

    for w in data:

        if w not in cnt.keys():
            cnt[w] = 1
        else:
            cnt[w] += 1

    if debug : print(cnt)

    return cnt
```

On lancera toutes nos procédures 20 fois, et on les sauvegardera dans un fichier binaire pour les utiliser plus tard. On a pour cette partie 1 : 

```python
if __name__ == '__main__':

    num_loop = 20
    num_proc_list = [1]
    
    document = loading()
    all_times = np.zeros((len(num_proc_list), num_loop))
    
    # seulement pour avoir une progress bar
    pbar = tqdm(total=num_loop*len(num_proc_list))

    for loop in range(num_loop):

        for i, num_proc in enumerate(num_proc_list):

            start_time = time()
            countMots(document, debug=False)
            end_time = time()

            all_times[i, loop] = end_time - start_time
            pbar.update(1)
	
    # on sauvegarde
    with open(f"mono_proc.dat", "wb") as f:
        pkl.dump({"num_proc":num_proc_list, "times":all_times}, f)
```

On obtient 5.93 sec, avec un écart-type de 0.17 sec.

## Partie 2 : Multi processeurs, sans *map reduce*

Dans cette partie, on va faire le même calcul que la partie 1, mais en parallélisant. On ajoute une fonction *decoupage_segments* qui permet de découper le texte en *num_proc* segments. Et on ajoute une fonction *countMotsMultiProc*, qui permet de faire le comptage parallèle.

```python
def decoupage_segments(data, num_proc):
    """
    Fonction qui permet de découper les data en num_proc segments
    """

    l = len(data)
    lpn = int(l/num_proc)
    
    segmenta = []

    for i in range(num_proc):

        segment = document[i*lpn:(i+1)*lpn] if i != num_proc-1 else document[i*lpn:]
        segmenta.append(segment)

    return segmenta
```

```python
def countMotsMultiProc(data, num_proc, debug=False):
	"""
	Parallélisation de countMots
	"""
    
    segments = decoupage_segments(data, num_proc)

    with Pool(num_proc) as p:
        resultat = p.map(countMots, segments)

    result = {}
    for d in resultat:
        for key, value in d.items():
            if key in result:
                result[key] += value
            else:
                result[key] = value

    if debug : print(result)
```

On teste pour les nombre de coeurs suivants : 2, 4, 6, 8, 10 (sur ma machine, j'ai 12 coeurs au total). On a le code suivant :

```python
if __name__ == '__main__':

    num_loop = 20
    num_proc_list = [2, 4, 6, 8, 10]
    
    document = loading()
    all_times = np.zeros((len(num_proc_list), num_loop))
    pbar = tqdm(total=num_loop*len(num_proc_list))

    all_times = np.zeros((len(num_proc_list), num_loop))
    pbar = tqdm(total=num_loop*len(num_proc_list))

    for loop in range(num_loop):

        for i, num_proc in enumerate(num_proc_list):

            start_time = time()
            countMotsMultiProc(document, num_proc, debug=False)
            end_time = time()

            all_times[i, loop] = end_time - start_time
            pbar.update(1)

    with open(f"multi_proc.dat", "wb") as f:
        pkl.dump({"num_proc":num_proc_list, "times":all_times}, f)
```

On comparera les résultats dans la partie **résultats**.

## Partie 3 : Multi processeurs, avec *map reduce*

On va enfin faire une partie en utilisant du *map_reduce*. 

On a d'abord une fonction *map_function* :

```python
def map_function(segment):
    """
    Fonction map qui permet d'obtenir une liste clé-valeurs de chaque mot du segment
    """

    map_words = [(word, 1) for word in segment]

    return map_words
```

Ensuite, il nous faut un shuffle : 

```python
def shuffle_function(maps_words, num_proc, rang='abcdefghijklmnopqrstuvwxyz'):
    """
    On trie les valeur pour les regrouper entre même groupe de lettre.
    Par exemple, si on a 3 proc, shuffled aura 3 liste de mots : les mots entre a et i, ceux entre i et r, et ceux entre r et z.
    """

    born_sup = list()

    for i in range(num_proc-1):

        born_sup.append(rang[int((i+1)*len(rang)/num_proc)])

    shuffled = [[] for _ in range(num_proc)]


    for map_words in maps_words:

        for word, count in map_words:

            letter = word[0].lower()

            find_proc = None

            for bi, born in enumerate(born_sup):
                if find_proc is None and letter < born:
                    find_proc = bi

            if find_proc is None : find_proc = num_proc - 1

            shuffled[find_proc].append((word, count))

    return shuffled
```

Après, il nous faut une fonction reduce :

```python
def reduce_function(word_counts):
    """
    Reduction des suffles en parallèle
    """
    return reduce(lambda acc, item: {**acc, item[0]: acc.get(item[0], 0) + item[1]}, word_counts, {})
```

Enfin, on agence ces fonctions, dans *parallel_word_count*, en parallélisant *map_function* et *reduce_function*, en appliquant entre deux un shuffle. On a :

```python
def parallel_word_count(words, num_proc, debug=False):
    """
    Fonction permettant de produire le map_reduce
    """

    # Découpage *num_proc* en segments
    segments = decoupage_segments(words, num_proc)

    # Parallèlisation du mappage
    with Pool(num_proc) as pool:
        mapped_results = pool.map(map_function, segments)
    
    # Shuffle
    shuffled_results = shuffle_function(mapped_results, num_proc)

    # Parallèlisation du reduce
    with Pool(num_proc) as pool:
        reduce_result = pool.map(reduce_function, shuffled_results)

    # On finis par concatener les reduce
    result = {}
    for d in reduce_result:
        result.update(d)
    
    # On prit que pour du debug
    if debug : print(result)
```

On execute ensuite le code : 

```python
if __name__ == '__main__':

    num_loop = 20
    num_proc_list = [2, 4, 6, 8, 10]
    
    document = loading()
    all_times = np.zeros((len(num_proc_list), num_loop))
    pbar = tqdm(total=num_loop*len(num_proc_list))

    all_times = np.zeros((len(num_proc_list), num_loop))
    pbar = tqdm(total=num_loop*len(num_proc_list))

    for loop in range(num_loop):

        for i, num_proc in enumerate(num_proc_list):

            start_time = time()
            parallel_word_count(document, num_proc, debug=False)
            end_time = time()

            all_times[i, loop] = end_time - start_time
            pbar.update(1)

    with open(f"map_reduce.dat", "wb") as f:
        pkl.dump({"num_proc":num_proc_list, "times":all_times}, f)
```

On peut passer au **résultats**.

## Partie 4 : Résultats

On peut regrouper tous nos résultats dans la figure suivante (l'image est aussi dispo dans le github au cas où ou à l'adresse https://drive.google.com/file/d/1sGYD1SDlzwzK6iQRulwbft331AdW5gpD/view?usp=sharing): 

![<image>](https://drive.google.com/file/d/1sGYD1SDlzwzK6iQRulwbft331AdW5gpD/view?usp=sharing)

On a trois courbes : 

* En vert, la référence à l'horizontal, qui est le comptage classique en mono-process. Il est en faire le plus rapide, avec 5,93 + = 0,17 secondes.
* Ensuite, en bleu, le comptage classique fait en parallèle. Les temps, quel que soit le nombre de coeurs, sont beaucoup plus grands que le comptage mono-process. Cependant, le temps diminue avec jusqu'à 6 coeurs, puis ré-augmente.
* Enfin, en rouge, on a utilisé la méthode *map-reduce*. Cependant, les temps sont plus élevés que les deux autres méthodes. Cet effet est peut-être lié au fait que la méthode est adaptée à de plus grandes données. Il faudrait effectuer ce test sur des données beaucoup plus volumineuses pour se faire un avis plus large. 

