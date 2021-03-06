# REPLICACIÓ via Binlog
## Master

Per començar hem d'entrar al fixer de configuració del mysql i activar els log binaris. Abans de fer qualsevol canvi es recomanable fer una copia de seguretat d'aquest fitxer. També hem d'afegir la línia server-id i un número per identificar el servidor. Al Slave també haurem de ficar el mateix (només el server-id) però amb un número diferent.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap1.png)

Ara hem d'apagar el servei i borrar tots els fitxers de log que té creats la base de dades dintre de /var/lib/mysql. Un cop que els tenim borrats, tornem a encendre el servei i podrem veure que ha creat de nous amb el nom que hem indicat abans.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap2.png)

Ara mirem el contingut dels logs binaris que s'han creat. És recomanable mirar sempre l'últim que s'ha creat. Hem de buscar la línia que fica "end_log_pos". En aquest cas (en la segona captura) ens indica que és el número 154.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap3%20cat%20rep%20archivo.png)

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap4%20cat%20rep%20archivo.png)

Ara hem de crear un usuari que servirà per fer les replicacions. Aquest usuari tindrà de nom slave i la ip de la màquina slave. També hem de donar permisos de replicació a aquest usuari.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap5%20usuari.png)

Per últim executem la comanda flush logs i comprovem que estan activats i creats correctament.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap8.png)




## Slave

A continuació configurarem l'slave.
Primer hem d'assignar un server id diferent de la màquina master.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap6%20slave.png)

Desde la màquina master, enviem un backup de la base de dades a la màquina slave.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap16.png)

Com que les màquines han de tenir la mateixa informació per començar amb la replica, importem les bases de dades que acabem d'enviar.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap9.png)

Obrim l'arxiu que fa de backup de la base de dades i busquem la línia on ens indica quin és l'últim log binari que té el master i en quina posició s'ha quedat. Normalment és troba per el principi d'aquest arxiu.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap17.png)

Ara hem d'executar les següents comandes, les quals tenen informació de la màquina master.

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap10.png)

I executar la comanda start slave:

![Captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap12.png)

Per comprovar si la màquina slave està ben configurada i està replicant del master ho podem comprovar amb la comanda show slave status \G;

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap13.png)

Ara comprovarem si la replica funciona creant unes taules en el servidor master.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap14.png)

Creem una taula, comprovem que esta en el servidor slave i creem una segona taula en el master i instantaneament comprovem que també s'ha creat en el servidor slave.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/cap15.png)

Nosaltres ens vam trobar amb un problema de replicació ja que com les màquines eren la mateixa (clonada) el mysql tenía el mateix id en els 2 servidors. Per tal de canviar un dels dos hem de borrar un arxiu que esta situat a /var/lib/mysql que es diu auto.cnf. Aquest guarda un ID per servidor. Per fer que es torni a crear hem de borrar aquest arxiu i reiniciar el servei.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/25.png)

# Replicació GTID

## Master

En el servidor que fa de Master tenim que habilitat el GTID amb la següent configuració, amb la opció de "enforce-gtid-consistency", que vol dir que fins que el Master no hagi fet el commit i que les dades estiguin dins de la BBDD el Slave no replicara la informació.
A part, també tindrem que tenir habilitat el binari log, que noslatres ja el tenim habilitat de l'activitat anterior.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/26.png)

## Slave

En el servidor que fara d'Slave, també tenim que habilitar el GTID.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/27.png)

Ara que ja el tenim actiu, reiniciem el servidor perquè hagafi les configuracions i ara el que farem sera desactivar totes les replicacions que tingui el servidor Slave. I posarem les noves dades del servidor Master a replicar.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/18.png)

Ara un cop que el tenim connectat, habilitem el proces de replica amb "start slave;" i també comprovem de que el proces estigui OK.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/19.png)

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/20.png)

Ara el que tenim que fer per asegurarnos de que esta llegint les dades correctament del servidor Master, és intriduir dades en la BBDD Master. En aquest cas hem creat la taula "c".

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/21.png)

Ara mirem que en el servidor Slave també l'ha creat

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/22.png)

Ara si tornem a executar l'estat de l'Slave veurem que al final ens indica l'últim ID que s'ha obtingut del Master i ell també ho ha executat.

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/23.png)

![captura](https://github.com/Shyrkoon/Base-de-dades/blob/master/Activitat4/img/24.png)




