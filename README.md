# meridian-api-product-analysis

Task 1 -What doesn't match?

I compared the API documentation with three responses provided. I found few places where the actual API behavior does not match what the documentation says.
1. Pagination issue - the biggest problem

   The documentation says the has_more tells the client whether another page of orders is available. If it is true, the client should use next_cursor to get the next page.
   In orders_page1.json, however, has_more is false, while next_cursor contains cur_8f2a19bd. The README shows that the cursor was actually used to retrieve orders_page2.json, which contains two more orders.
   This could cause a client to stop after page1 and never retrieve those two orders. page1 adds upto $249.94, while page 2 adds another $79.09. So an intergration following the documentation could silenlty incomplete numbers.
   I consider this the most serious issue because it can affect the whole dataset without causing an obvious error.

2. Refunded is not documented as a status

   The documentation lists four possible statuses: pending, shipped, delivered, and cancelled.
   But in orders_page2.json contains ord_1003 with status refunded.
   This could cause problem for a system that checks the status against the documented list. It is also important for finance because a refunded order may need to be treated differently from a normal completed order.

3. Money is not always returned in the documented format
   
   The documentation says that monetary values are integers in the smallest currency unit. For example, $54.70 should appear as 5470.
   But in orders_page2.json, ord_1006 has:
   I.subtotal: 44.0
   II.tax: 3.63
   III.shipping: 5.9
   IV.total: 53.62
   The calculation itself is correct: $44.00 + $3.63 + $5.99 = $53.62. The problem is that the format does not follow the documentation.
   This could be session for an integration expecting cents. For example, it might interpret 44.0 as 44 cents instead of $44.00

4. Customer email can be missing
   
   The documentation says the customer's email is always present.
   In orders_page2.json, ord_1005.customer.email is null.
   This could cause problem for systems that depend on an email being available for customer matching, notifications, or other processing.

5. A missing order returns 200 instead of 404
   
   The documentation says that requesting an order that does not exist should return HTTP 404.
   The request for ord_9999 instead returned HTTP 200 with:
   {"order": null}
   A client may interpret the 200 response as a successful request and not handle the missing order correctly.

Task 2 - What's the total revenue?

Adding the total field from all six captured orders gives:
$328.03
However, I would be careful about calling this the actual recognized revenue.
One order, ord_1003, has a refunded status, but the API does not tell us how much was refunded or how refunds should be treated in the revenue calculation.
There is also the pagination issue. If a client stopped after page 1 because has_more was false, it would calculate $249.94 and miss $79.90 from page 2.
So my conculsion is: 
Raw sum all captured order totals: $328.03.
To calculate the actual revenue used by the dashboard, I would want to know how the dashboard treats refunded orders and whether these six orders represents the complete period being compared.

Task 3A - Reply to Priya

Subject Re: Revenue reconciliation

Hi Priya,

I found a pagination issue that could explain part of the difference.
The API documentation says to stop requesting pages when has_more is false. However, the first response says has_more is false even though there is another page of orders.
If your integration followed the documentation, it would only process the first four orders, totaling $249.94, and miss another $79.09 from the second page.
The raw total across all six orders is $328.03. There is also an order marked as refunded, but the API doesn't explain how that should be treated in revenue, so I would confirm that with the dashboard definition before calling $328.03 the final revenue figure.

Best regards,

Gaganshekar C

Task 3B - Bug report

Title: Orders API reports has more=false when another page exists

what I found:

orders_page1.json returns has more=false but also provides next_cursor=cur_8f2a19bd. Using this cursor returns orders_page2.json, which contains two additional orders.

what should happen:

has_more should be true whenever more orders are available, and next_cursor should be provided for the next request.

what happens now:

The API tells clients to stop even though more orders exist.

Impact

an integration following the documentation can silently miss orders and produce incorrect financial reports.

what to check:

Review the pagination logic that calculates has_more and next_cursor, especially the logic determining whether additional records remain.

