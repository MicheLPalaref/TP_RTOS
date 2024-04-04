# TP_RTOS
# 1 FreeRTOS, Taches et Semaphores

# 1.1 Tache simple
1)
En quoi le parametre TOTAL HEAP SIZE a de l'importance: TOTAL_HEAP_SIZE est le parametre qui defini la quantite de RAM disponible pour le tas du RTOS. Il defini la quantite de RAM disponible pour les variables dynamiques.

Par defaut la quantite de RAM allouee est faible. Elle peut etre augmentee jusqua atteindre 100% de la RAM. Mais ce faisant, il nous sera impossible de creer des variables globales par manque d'espace.

2)
La macro portTICK_PERIOD_MS permet de calculer le temps reel en fonction de la frequence de tic, avec une resolution d'un tic par periode.

# 1.2 Semaphores pour la synchronisation
6)
Lorsque l'on met TaskGive prioritaire par rapport a TaskTake, l'ordre d'apparition des prints dans l'hyperterminal apres initialisation donne: 
AvantGive, ApresGive, Avant Take, ApresTake, AvantTake. 

Lorsque l'on met TaskTake prioritaire par rapport a TaskGive, l'ordre d'apparition des prints dans l'hyperterminal apres initialisation donne: 
AvantTake, AvantGive, ApresTake, Avant Take, ApresGive. 

Ces changements s'expliquent par le fait que lorsque le TaskTake est prioritaire, il doit attendre le semaphore du TaskGive avant de passer et se met en pause (laissant passer le TaskGive). Lorsque le TaskGive du semaphore passe, le TaskTake reprend le dessus puisqu'il est prioritaire sur le TaskGive.

# 1.3 Notifications
7)
On remplace 

xSemaphoreGive(xSemaphore);

par 

xTaskNotifyGive(handle_taskTake);

dans TaskGive et on remplace 

if (xSemaphoreTake(xSemaphore, 1000)== pdTRUE){
		printf("Apres TaskTake\r\n");} 

par

if (ulTaskNotifyTake(pdTRUE,1000)== pdTRUE){
		printf("Apres TaskTake\r\n");}

dans TaskTake.

# 1.4 Queues
8)
on cree la queue 
QueueHandle_t xQueue;//declaration de la variable
xQueue = xQueueCreate(5, sizeof(int)); //Dans le main

Puis dans TaskGive:
if (xQueue !=0){
	xQueueSend( xQueue, (const void *) &i, portMAX_DELAY);}

Puis dans TaskTake:
if (xQueueReceive( xQueue, (void *) &reception, 1000)== pdTRUE){
	printf("Apres TaskTake %d\r\n", reception);}
