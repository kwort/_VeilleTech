Event
=====

...

raw
---

Pendant l'exécution d'une application Symfony, de nombreuses notifications d'événements sont déclenchées. Votre application peut écouter ces notifications et y répondre en exécutant n'importe quel code.

Les événements internes fournis par Symfony eux-mêmes sont définis dans la classe KernelEvents. Les bundles et les bibliothèques tierces déclenchent également de nombreux événements et votre propre application peut déclencher des événements personnalisés.

Tous les exemples illustrés dans cet article utilisent le même événement KernelEvents::EXCEPTION pour des raisons de cohérence. Dans votre propre application, vous pouvez utiliser n'importe quel événement et même mélanger plusieurs d'entre eux dans le même subscriber.

### Creating an Event Listener

La manière la plus courante d'écouter un événement est d'enregistrer un event listener:

```php
<?php
// src/AppBundle/EventListener/ExceptionListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;

class ExceptionListener
{
    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        // You get the exception object from the received event
        $exception = $event->getException();
        $message = sprintf(
            'My Error says: %s with code: %s',
            $exception->getMessage(),
            $exception->getCode()
        );

        // Customize your response object to display the exception details
        $response = new Response();
        $response->setContent($message);

        // HttpExceptionInterface is a special type of exception that
        // holds status code and header details
        if ($exception instanceof HttpExceptionInterface) {
            $response->setStatusCode($exception->getStatusCode());
            $response->headers->replace($exception->getHeaders());
        } else {
            $response->setStatusCode(Response::HTTP_INTERNAL_SERVER_ERROR);
        }

        // Send the modified response object to the event
        $event->setResponse($response);
    }
}
?>
```

Chaque événement reçoit un type légèrement différent de l'événement $event. Pour l'événement kernel.exception, il s'agit de GetResponseForExceptionEvent. Pour voir quel type d'objet chaque écouteur d'événement reçoit, voir KernelEvents ou la documentation sur l'événement spécifique que vous écoutez.

Maintenant que la classe est créée, il suffit de l'enregistrer en tant que service et de notifier Symfony qu'il s'agit d'un "listener" sur l'event kernel.exception en utilisant une balise "tag":

```yaml
# app/config/services.yml
services:
    app.exception_listener:
        class: AppBundle\EventListener\ExceptionListener
        tags:
            - { name: kernel.event_listener, event: kernel.exception }
```

Il existe un attribut tag facultatif appelé "method" qui définit la méthode à exécuter lorsque l'événement est déclenché. Par défaut, le nom de la méthode + "camel-cased event name". Si l'événement est kernel.exception, la méthode exécutée par défaut est onKernelException().

L'autre attribut tag facultatif est appelé priority, dont la valeur par défaut est 0 et qui contrôle l'ordre dans lequel les auditeurs sont exécutés (plus la priorité est élevée, plus un écouteur est exécuté). Ceci est utile lorsque vous devez garantir qu'un auditeur est exécuté avant un autre. Les priorités des auditeurs internes de Symfony varient habituellement de -255 à 255, mais vos propres auditeurs peuvent utiliser n'importe quel entier positif ou négatif.

### Creating an Event Subscriber

Une autre façon d'écouter des événements est par l'intermédiaire d'un event subscriber, qui est une classe qui définit une ou plusieurs méthodes qui écoutent un ou plusieurs événements. La principale différence avec les auditeurs de l'événement est que les abonnés savent toujours quels événements ils écoutent.

Dans un event subscriber donné, différentes méthodes peuvent écouter le même événement. L'ordre dans lequel les méthodes sont exécutées est défini par le paramètre de priorité de chaque méthode (plus la priorité est élevée, plus la méthode est appelée). Pour en savoir plus sur les event subscriber, lisez le composant EventDispatcher.

L'exemple suivant montre un event subscriber qui définit plusieurs méthodes qui écoutent le même événement kernel.exception:

```php
<?php
// src/AppBundle/EventSubscriber/ExceptionSubscriber.php
namespace AppBundle\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpKernel\KernelEvents;

class ExceptionSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        // return the subscribed events, their methods and priorities
        return array(
           KernelEvents::EXCEPTION => array(
               array('processException', 10),
               array('logException', 0),
               array('notifyException', -10),
           )
        );
    }

    public function processException(GetResponseForExceptionEvent $event)
    {
        // ...
    }

    public function logException(GetResponseForExceptionEvent $event)
    {
        // ...
    }

    public function notifyException(GetResponseForExceptionEvent $event)
    {
        // ...
    }
}
?>
```

Il vous suffit d'enregistrer la classe en tant que service et d'ajouter la balise kernel.event_subscriber pour indiquer à Symfony qu'il s'agit d'un event subscriber:

```yaml
app/config/services.yml
services:
    app.exception_subscriber:
        class: AppBundle\EventSubscriber\ExceptionSubscriber
        tags:
            - { name: kernel.event_subscriber }
```

### Request Events, Checking Type

Une seule page peut faire plusieurs requêtes (une requête maître, puis plusieurs sous-requêtes, généralement sous la forme Comment intégrer des contrôleurs dans un modèle). Pour les événements Symfony de base, vous devrez peut-être vérifier si l'événement concerne une requête "maître" ou une "sous-requête":

```php
<?php
// src/AppBundle/EventListener/RequestListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\HttpKernel;
use Symfony\Component\HttpKernel\HttpKernelInterface;

class RequestListener
{
    public function onKernelRequest(GetResponseEvent $event)
    {
        if (!$event->isMasterRequest()) {
            // don't do anything if it's not the master request
            return;
        }
        // ...
    }
}
?>
```

Certaines choses, comme vérifier l'information sur la demande réelle, peuvent ne pas avoir besoin d'être faites sur les listeners de sous-requête.

### Listeners or Subscribers

Les listeners and subscribers peuvent être utilisés dans la même application indistinctement. La décision d'utiliser l'un d'eux est généralement une question de goût personnel. Cependant, il ya quelques avantages mineurs pour chacun d'eux:

- Les subscribers sont plus faciles à réutiliser car la connaissance des événements est conservée dans la classe plutôt que dans la définition du service. C'est la raison pour laquelle Symfony utilise les subscribers en interne;
- les listeners sont plus flexibles car les ensembles peuvent activer ou désactiver chacun d'eux conditionnellement en fonction de la valeur de configuration.

### Debugging Event Listeners

You can find out what listeners are registered in the event dispatcher using the console. To show all events and their listeners, run:

```bash
php bin/console debug:event-dispatcher
```

You can get registered listeners for a particular event by specifying its name:

```bash
php bin/console debug:event-dispatcher kernel.exception
```

