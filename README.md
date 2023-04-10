Используется контейнер arturakh/k8s:v3 - образ nginx, включает в себя страницу с информацией IP, имя хоста и утилиты для диагностики сети.

1) у нас мультизональный кластер (три зоны), в котором пять нод:

создаем облачный кластер например в GKE: 

gcloud container clusters create staging-cluster \
    --zone europe-west3 \
    --node-locations europe-west3-a,europe-west3-b,europe-west3-c \
    --num-nodes=5 

2) приложение требует около 5-10 секунд для инициализации:

можно добавить readinessProbe. Параметр initialDelaySeconds: 10. 
Все это время на под не будет поступать трафик.

3) по результатам нагрузочного теста известно, что 4 пода справляются с пиковой нагрузкой:

можно добавить HorizontalPodAutoscaler, укажем 4 реплики максимум и минимум 2.

4) на первые запросы приложению требуется значительно больше ресурсов CPU, в дальнейшем потребление ровное в районе 0.1 CPU. По памяти всегда “ровно” в районе 128M memory:

добавил resources. Минимальное CPU 100m, лимит 200m. Память всегда 128mi.

5) приложение имеет дневной цикл по нагрузке – ночью запросов на порядки меньше, пик – днём:

используем HorizontalPodAutoscaler, который будет уменьшать кол-во подов, когда отсутвует нагрузка на ЦП.

6) хотим максимально отказоустойчивый deployment:

-используем разные зоны для нодов. Используем HorizontalPodAutoscaler и readinessProbe.
-можно добавить liveness probe, чтобы он перезапускал под, если не будут пройдены проверки
-настроить VerticalPodAutoscaler и автоматическое масштабирование воркер нод, так же можно добавить еще мастер нод
-настроить мониторинг и логирование.

7) хотим минимального потребления ресурсов от этого deploymentа:

Настроили resources.
