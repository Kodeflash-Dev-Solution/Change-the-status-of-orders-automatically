# Change-the-status-of-orders-automatically

One way to keep your online store clean and organized is by using the order status. Complete, Cancelled, Awaiting Payment are statuses which helps you to pay attention to the most important orders.

But when you have a lot of “Awaiting payment” orders and you don’t know until when you need to consider that order?

I want to cancel my pending orders (awaiting payment) after 4 days.

Before the solution we discuss about:
- Should we need to consider all the payment methods?
- The client should receive any type of email confirmation?
- Which order status we need to consider?

1. Create a query with WooCommerce orders.
```php
$query = ( 
 array(
  'limit'   => 5,
  'orderby' => 'date',
  'order'   => 'DESC',
  'status'  => array( 'pending' )
 ) 
);

$orders = wc_get_orders( $query );

// our orders loop
foreach( $orders as $order ){		
		
}

// For performance reasons, we only 5 orders each time I run my snippet.
```

2. Check the number of days between order name and today
```php
//Inside the loop I wrote above, we will check the time difference and know the “age” of your order

$query = ( 
 array(
  'limit'   => 5,
  'orderby' => 'date',
  'order'   => 'DESC',
  'status'  => array( 'pending' )
 ) 
);

$orders = wc_get_orders( $query );

foreach( $orders as $order ){		
  
  // date of our order
  $date = new DateTime( $order->get_date_created() );
  // today's date
  $today = new DateTime();
  
  $interval = $date->diff($today);
  // days between order name and today	
  $datediff = $interval->format('%a');
   
  // 4: minimum of days the order can be considered
  if( $datediff >= 4 ){
    $order->update_status('cancelled', 'Cancelled for missing payment);
   }
}

// So far If the order is pending (as we defined on the query) and it has more than 4 days, we will update the order status to “canceled“, adding a status note (Cancelled for missing payment).
```

3. Tell to WordPress when to run our function
```php
// Add code into a function or simmilar:

function autocancel_wc_orders(){
  $query = ( array(
    'limit'   => 5,
    'orderby' => 'date',
    'order'   => 'DESC',
    'status'  => array( 'pending' )
  ) );
	$orders = wc_get_orders( $query );
	foreach( $orders as $order ){		

		$date     = new DateTime( $order->get_date_created() );
		$today    = new DateTime();
		$interval = $date->diff($today);
	
		$datediff = $interval->format('%a');

		if( $datediff >= 4 ){
			$order->update_status('cancelled', 'Cancelled for missing payment');
		}
	}
}

// Now we just need to add an action to run this. In this case, I use the simple admin_init action but alternatively, you could use like WPcron.
add_action( 'admin_init', 'autocancel_wc_orders' );
```
