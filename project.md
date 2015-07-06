
<h2> Formal Methods Project: "Crossroads" </h2>
<h5> Nika Pona, EMCL 2014/2015, ID: 13025 </h5>

I chose to model the traffic lights controller specified in the attached file *crossroads.pdf*. My program is contained in the file *project.smv*. It consists of two modules: the obligatory **main** module and the **controller** module called from **main** to create states that specify West-East (*we*) and North-East (*ns*) controllers respectively. Intuitively, each controller is a function from other road's state and their traffic lights that represents the possible situations on each road.  

~~~prolog
MODULE main
VAR
we :  controller(ns.state,ns.green);
ns :  controller(we.state,we.green);
ASSIGN
init(ns.green) := TRUE;
init(we.green) := FALSE;
~~~

In the module **controller** I define the possible states of the road and traffic lights following the specifications. This program is quite modular: if desired one can easily add other states representing further possible states of the road.

~~~prolog

MODULE controller(other_state,other_green)
VAR
state : {detected,waiting,passing};
green : boolean;
ASSIGN
next(state) :=
case
  state in {detected,waiting} & green : passing;
  state in {detected,waiting} & !green: waiting;
  state = passing : detected;
esac;
next(green) :=
case
  state = detected & other_state in {detected,waiting} & other_green : FALSE;
  state = detected : TRUE;
  state = passing & other_state = passing & other_green : TRUE;
  state = passing : FALSE;
  state = waiting & other_state in {detected,waiting} & other_green: FALSE;
  state = waiting : TRUE;
esac;

~~~

The following specifications are true in the system:

* If there is a car waiting on the *West-East* road, it will eventually pass:

```prolog
AG (we.state = waiting -> AF we.state = passing)
```
* If there is a car waiting on the *North-South* road, it will eventually pass:

```prolog
AG (ns.state = waiting -> AF ns.state = passing)
```

* It is never the case that both traffic lights are green:

```prolog
AG !(ns.green = TRUE & we.green = TRUE)
```

*  If there are cars waiting on both roads, one of them will pass in the next moment:

```prolog
AG (we.state = waiting & ns.state = waiting -> AX (we.state = passing | ns.state = passing) )
```
* Light will not be red all the time:

```prolog
AG AF (we.green = TRUE)
```

* If the car is waiting, then it will be waiting until the lights turned green:

```prolog
AG (ns.state = waiting -> A [ns.state = waiting U ns.green = TRUE ])
```
