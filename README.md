# TP_RTOS
# 1 FreeRTOS, Taches et Semaphores

# 1.1 Tache simple
1)
En quoi le parametre TOTAL HEAP SIZE a de l'importance: TOTAL_HEAP_SIZE est le parametre qui defini la quantite de RAM disponible pour le tas du RTOS. Il defini la quantite de RAM disponible pour les variables dynamiques.

Par defaut la quantite de RAM allouee est faible. Elle peut etre augmentee jusqua atteindre 100% de la RAM. Mais ce faisant, il nous sera impossible de creer des variables globales par manque d'espace.

2)
La macro portTICK_PERIOD_MS permet de calculer le temps reel en fonction de la frequence de tic, avec une resolution d'un tic par periode.

# 1.2 Semaphores pour la synchronisation

