- title : Building microservices with F#
- description : A journey into the microservice world with F#, Net Core and Docker
- author : Alexander Mogilka
- theme : moon
- transition : default

***
## Building microservices with F#
#### A journey into the microservice world with F#, Net Core and Docker
<br />
Alexander Mogilka
<br />
[@alxmglk](http://www.twitter.com/alxmglk)

***
## Let's start with a monolith

<img src="images/monolith.png" style="background: transparent; border-style: none;"  />

---
## Limitations and possible issues
* it’s difficult to distribute responsibility across the team members and manage release cycle as project and team size grow
* you ought to retest the entire system even if you did only a small change
* it's not possible to scale the system efficiently
* increased fragility since error in a single module might kill the entire system

---
## But the monolith is a good way to start out!
https://martinfowler.com/bliki/MonolithFirst.html

***
## Microservices to the rescue
<img src="images/microservices.png" style="background: transparent; border-style: none;"  />

<small>Small autonomous services that work together and modelled around a business domain</small>

---
## Benefits of the microservices
* independent release cycle of each service
* failures are isolated
* granular scaling
* freedom to choose the tech stack suitable for particular task

---
## How about challenges?
#### Welcome to <strike>hell</strike> the world of distributed systems
* everything fails all the time
* additional overhead of the remote calls
* take care of the inter-microservice contracts
* increased operational costs

***
## Reduce the impact of failures
* don't ever use synchronous calls
* fail fast
* minimize allotted timeouts
* make failures observable

***
## Synchronous calls are pure evil
A bunch of parallel synchronous calls will suddenly exhaust the thread pool

<img src="images/microservices-synch-and-async-calls.png" style="border-style: none;" width="70%"  />

---
## Writing async code in F# is a piece of cake
    // MerchantId -> Async<MerchantDiscount>
    let getMerchantDiscount merchantId = ...
    // ProductId -> MerchantDiscount -> Async<ProductPrice>
    let getProductPrice productId discount = ...

    // 1st approach : async workflow
    // Async<ProductPrice>
    async {
        let! discount = getMerchantDiscount merchantId
        return! getProductPrice productId discount
    }
    // 2nd approach: more idiomatic way
    // MerchantId -> Async<ProductPrice>
    getMerchantDiscount >> Async.bind (getProductPrice productId)

***
## Why it's important to fail fast
Slow failures propagate from the dependencies up to the consumers

<img src="images/microservices-slow-failures.png" style="border-style: none;" width="60%"  />

---
## Circuit breaker
<img src="images/circuit-breaker.png" width="350px" style="background: transparent; border-style: none;"  />

[https://martinfowler.com/bliki/CircuitBreaker.html](https://martinfowler.com/bliki/CircuitBreaker.html)

---
## Circuit breaker and retries in the wild
    type AsyncArrow<'a,'b> = 'a -> Async<'b>
    
    // AsyncArrow<Guid, HttpResponseMessage>
    let getProductPrice productId = ...

    // AsyncArrow<Guid, HttpResponseMessage> - the signature is still the same
    let execute = 
        getProductPrice
        |> AsyncArrow.after (updateInvoice invoice)
        |> AsyncArrow.retry retryCount backoffStrategy
        |> AsyncArrow.circuitBreaker circuitBreakerPolicy

***
## Make failures discoverable
* collect and aggregate logs
    * don't forget about correlation ids
* collect and aggregate metrics
* monitoring

---
## Seamless incorporation of the logging
    let logStart _ = log.Info "Import started"
    let logFinish _ _ = log.Info "Import finished"
    let logError ex = 
        sprintf "An error has occured during the import: %s" ex.Message 
        |> log.Error 

    importProducts
    |> updateInventory
    |> AsyncArrow.before logStart
    |> AsyncArrow.after logFinish
    |> AsyncArrow.onError logError

---
## Correlation Ids
<img src="images/microservices-correlation-id.png" style="background: transparent; border-style: none;"  />

---
## Inject correlation id into the service request
    // HttpRequestMessage -> Async<HttpResponseMessage>
    let makeHttpRequest = ...

    // HttpRequestMessage -> HttpRequestMessage
    let injectCorrelationId correlationId (req : HttpRequestMessage) =
        req.Headers.Add ("Correlation-Id", correlationId)
        req

    // HttpRequestMessage -> Async<HttpResponseMessage>
    let makeHttpRequestWithCorrelationId = 
        makeHttpRequest 
        |> AsyncArrow.mapIn (injectCorrelationId correlationId)

***
## API Evolution
* strangler pattern for evolving API
* Postel's law
* write consumer tests on the API and run them on each check in of the producer
* avoid non-explicit serialization

***
## Functional composition is a powerful technique

Due to rich capabilities of functional composition you could easily address cross-cutting concerns (retries, timeouts, logging etc) without any changes to your business logic

***
## Embrace the culture of automation
* each microservice describes its own build/deploy pipeline
    * Jenkins Pipeline + Blue Ocean plugins + Jenkinsfile
* containerization and clusterization
* dashboard for monitoring the microservices
* 

***
## Conclusions
* Microservices architecture gives you a lot of perks but in the same time requires a decent level of expertise for the team
* F# and functional approach work greate for the microservices
* 

***
## Questions?