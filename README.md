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
```C
xSemaphoreGive(xSemaphore);
```
par 
```C
xTaskNotifyGive(handle_taskTake);
```
dans TaskGive et on remplace 
```C
if (xSemaphoreTake(xSemaphore, 1000)== pdTRUE){
		printf("Apres TaskTake\r\n");} 
```
par
```C
if (ulTaskNotifyTake(pdTRUE,1000)== pdTRUE){
		printf("Apres TaskTake\r\n");}
```
dans TaskTake.

# 1.4 Queues
8)
on cree la queue 
```C
QueueHandle_t xQueue;//declaration de la variable
xQueue = xQueueCreate(5, sizeof(int)); //Dans le main
```
Puis dans TaskGive:
```C
if (xQueue !=0){
	xQueueSend( xQueue, (const void *) &i, portMAX_DELAY);}
```
Puis dans TaskTake:
```C
if (xQueueReceive( xQueue, (void *) &reception, 1000)== pdTRUE){
	printf("Apres TaskTake %d\r\n", reception);}
```
# 1.5 Reentrance et exclusion mutuelle
11)
Dans la console, les taches 1 et 2 s'endorment toutes les 2 pour 2 ticks.
Cela est du au printf de la tache 1 qui ne s'effectu pas suffisament rapidement. Il est toujours interrompu par la tache 2. La fin du printf de la tache 1 est donc remplace par le printf de la tache 2.

Pour modifier cela, on met en place un semaphore mutex qui permet de bloquer les interruptions eventuelles pour proteger les printf des taches 1 et 2.
En pratiquen on appelle le semaphoreTake mutex avant un printf et on appelle un semaphoreGive mutex apres un printf.

# 2 On joue avec le Shell


![image](https://github.com/MicheLPalaref/TP_RTOS/assets/144770627/da115db0-ce1c-400c-85b2-d005a1e31565)

Les premieres fonctions appellent une fonction bidon ou une fonciton somme.

Pour creer la fonction Led, cela se deroule en plusieurs etapes:
La tache Task_blink_led:

```C
	//Le handler
TaskHandle_t handle_task_blink_led;

	//la tache
static uint32_t period = 100; // variable globale statique

void task_blink_led(int * unused){

	vTaskSuspend(0);	// Se suspend elle meme

	for( ;; )
	{
		/* Simply toggle the LED every period ms, blocking between each toggle. */
		HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
		//vToggleLED();
		vTaskDelay( period / portTICK_PERIOD_MS );
	}
}

	//Task Create
xTaskCreate(
		task_blink_led,
		"LED",
		256,
		NULL,
		1,
		&handle_task_blink_led
	);
```

Ensuite on cree le shell led

```C
int led(int argc, char ** argv)
{
	/*fait clignoter P1, un param gère la periode de developpement, 0=eteint, le cligno se fait dans une tache*/

	printf("Je suis une fonction qui allume une led \r\n");

	period = atoi(argv[1]);
	if (period == 0) {
		printf("LED OFF\r\n");
		// suspend la tache
		// eteind la led
		vTaskSuspend(handle_task_blink_led);
		HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET);
	}
	else
	{
		printf("LED BLINK\r\n");
		// clignote la led
		// resume la tache
		vTaskResume(handle_task_blink_led);
	}

	return 0;
}

void task_shell(void * unused)
{
	shell_init();
	shell_add('f', fonction, "Une fonction inutile");
	shell_add('a', addition, "Effectue une somme");
	shell_add('l', led, "Led Clignotte");
	shell_run();	// boucle infinie
}

```
![alt text](image.png)

en appelant le shell led et en definissant son parametre "period" on fait varier la periode de clignottement de la led P1. Si period = 0, la led s'eteind.


# 3 Debug, gestion d'erreur et statistiques

# 3.1 Gestion du tas
1.
Segment de donnees: zone memoire dreservee a l'allocation des variables statiques (globales, globales static, locales static)

Pile: Variables locales (temporaires) = un genre d'allocation dynamique, mais different du malloc
Quand on cree une variable dans une fonction, elle est cree dans la pile. Quand on sort de la fonction, les variables disparaissent.

Tas: zone memoire reservee a l'allocation des variables dynamiques (malloc ou x*Create).

2.
La zone reservee a l'allocation dynamique est geree par FreeRTOS.

3.

Il existe 2 facons de signaler une erreur:
```C
//premiere facon:
if (xTaskCreate(task_shell, "Shell", TASK_SHELL_STACK_DEPTH, NULL, TASK_SHELL_PRIORITY, &h_task_shell) != pdPASS)
	{
		printf("Error creating task shell\r\n");
		Error_Handler();
	}

// Deuxieme facon:
	BaseType_t ret;

	ret = xTaskCreate(
			task_blink_led,
			"LED",
			256,
			NULL,
			1,
			&handle_task_blink_led
	);

	if (ret != pdPASS)
	{
		printf("Error creating task LED\r\n");
		Error_Handler();
	}

```
4.
![alt text](image-1.png)

La memoire RAM utilisee est de 320 Ko.

La memoire FLASH utilisee est de 1024 Ko

5.
Pour creer des taches jusqu'a obtenir une erreur, on implemente une boucle FOR sur le TaskCreate:
![alt text](image-2.png)

6.
On remarque que:

La memoire RAM utilisee est de 320 Ko.

La memoire FLASH utilisee est de 1024 Ko
Soit, aucun changement avec le cas precedent.

7.
Pour changer la valeur du tas, on change la valeur TOTAL_HEAP_SIZE de FreeRTOS d'un facteur 10:
![alt text](image-3.png)

La nouvelle taille de la memoire est donc:
![alt text](image-4.png)
![alt text](image-5.png)

8.
On remarque que la RAM utilisee est passee de 5.73% a 47.92%
ceci est la consequence de l'augmentation de la valeur TOTAL_HEAP_SIZE.
Le nombre de taches pouvant etre execute passe de 7 a 71.



# 3.2 Gestion des piles

![alt text](image-6.png)

On mets egalement RECORD_STACK_HIGH_ADDRESS a enable
![alt text](image-7.png)


Pour creer une erreur sur la pile, on creer dans le task_sheel:
```C
	int tableau[512];

	for (int i = 0 ; i < 512 ; i++)
	{
		tableau[i] = i;
		printf("%d %d\r\n", i, tableau[i]);
	}

	printf("Zut\r\n");



//En amont on defini la tache vApplicationStackOverflowHook qui sera appelee a chaque depassement de la pile
	void vApplicationStackOverflowHook(xTaskHandle xTask, signed char *pcTaskName)
{
	/* Run time stack overflow checking is performed if
   configCHECK_FOR_STACK_OVERFLOW is defined to 1 or 2. This hook function is
   called if a stack overflow is detected. */
	HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);//Allumer la LED
	Error_Handler();//Appel du ERROR_HANDLER

}
```


Idle Hook Function sert a mettre le processeur en economie d'energie.
Il est appele par les taches de derineres priorite, n'etant donc appele que si toutes les autres fonctions ne sont pas appelees.

Tick Hook Function sert a implementer des fonctionnalites de timer.

Malloc Failed Hook Function sert a gerer les problemes causes par des malloc.

# 3.3 Statistiques dans l'IDE
1.
![alt text](image-8.png)

4.
En lancant puis en mettant en pause, la fenetre FreeRTOS Task List affiche les taches suivantes:
![alt text](image-9.png)

Dans une tache avec peu de variables locales, le Shell ne prends que 18% de la pile:
![alt text](image-10.png)

Dans une tache avec 128 variables locales, le Shell prends 52.7% de la pile:
![alt text](image-11.png)



6.
On utilise les deux fonctions suivantes:
```C
void configureTimerForRunTimeStats(void)
{
	// Appelée au début par l'OS
	// Lance un timer
	HAL_TIM_Base_Start(&htim2);
}

unsigned long getRunTimeCounterValue(void)
{
	// Appelé à chaque changement de contexte
	// Il faut retourner la valeur du compteur
	return __HAL_TIM_GET_COUNTER(&htim2);
}
```
Puisqu'on va utiliser le timer2, on doit l'activer:
![alt text](image-12.png) 
On oubliera pas de parametrer le Prescaler a 10 ou 100 fois la vitesse d'execution de l'OS (108 000 000 Hz)

Lorsqu'on utilise vTaskDelay, l'Idle est utilise presque 100% du temps puisque les taches LED et Shell ne sont appelees qu'a des moments disctincts et que l'IDLE tourne en boucle.
![alt text](image-14.png)


Lorsqu'on utilise HAL_Delay, on voit la tache LED etre utilisee de plus en plus avec le temps si la LED est appelee.
Au final, on peut se rendre compte du temps pendant laquelle chaque tache se deroule:
![alt text](image-16.png)

![alt text](image-17.png)

Si la tache LED tourne, LED aura plus de temps d'occupation de la memoire.
Si la tache LED ne tourne pas, IDLE aura plus de temps d'occupation de la memoire.

Cet equilibre evolu en fonction du temps ou chaque fonction est active.
