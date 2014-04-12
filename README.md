API with Guaranteed Delivery
============================

This example exposes an API, supposedly to receive orders from a mobile device, acknowledge the receipt of the same and then forward the request to the backend Order Processing system (in this case a database). AMQP is used as Message Broker.

Points to Note
==============

	1. AMQP has ackMode set to Manual. This is because AMQP cannot participate in an XA-Transaction with the Database.
	2. According to the Protocol, we publish to an AMQP exchange using a routing key. We have no knowledge of teh destination Queue. The binding between the routing key and the destination queue is done on AMQP server admin. We consume from the known Queue.
	3. Just like JMS, we can set attributes (headers) on the Message before publishing. We do this in the normal way in Mule with the set-property transformer.
	4. We surround the database inserts and the explicit AMQP Acknowledge with a Transactional scope. If any of these fail, we'll get a Rollback on the database, and an Exception is thrown.
	5. The Rollback-Exception-Strategy will catch the above Exception and Reject the AMQP Message, thus leaving it on the Queue. On redelivery exhaustion, we Accept the Message and forward it to a Dead letter Queue (again by way of a routing key).
	6. The Database interaction is done by capturing the auto-generated primary key on the Order (set to flowVars.orderId) and then using this for the inserts for each order item. The latter are done in bulk mode. No need for a for-each. The Database connector will iterate through each item in our List and send in bulk an sql insert statement for each of these.

Contact
=======
nial.darbey@mulesoft.com