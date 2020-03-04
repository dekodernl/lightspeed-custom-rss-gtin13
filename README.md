# Lightspeed Custom RSS GTIN-13 with calculated check digit
When you need to convert a Lightspeed RSS feed product id that is not 
compatible with a GTIN-13 code, for example for import into Facebook,
conversion to this is not available by default. You can convert this
using this code.

### What is a GTIN-13?
This is an international product-id format. The product-id contains of
12 arbitrary selected numbers and a 13th check-digit. This check-digit 
is calculated by multiplying every odd digit by 1 and every even digit 
by 3. The sum of these multiplied digits is subtracted from the closest 
higher number devidable by 10. For instance if the sum is 57 this is 
subtracted from 60 thus 3 is the check-digit.

### How does the code-conversion work?
Here is a step-by-step walk thru so you can learn and adapt if necessary.

Within the loop `{% for product in products %}` the `product.id` property
is available.

```
{% for product in products %}
// For example:
// product.id = 12345678

// First make sure the product-id is 12 digits long:
// Reverse the string and append 12 zeros to the end
{% set reverse_product_id = product.id | reverse ~ "000000000000" %}
// reverse_product_id is now 87654321000000000000

// Select the first 12 digits and reverse the string again 
{% product_id_12_long = reverse_product_id[:12] | reverse %}
// product_id_12_long is now 000012345678

// Determination of odd / even not available. Therefor manually
// selecting and multiplying each digit with either 1 or 3
{% set d1  = product_id_12_long[0:1]  * 1 %}
{% set d2  = product_id_12_long[1:1]  * 3 %}
{% set d3  = product_id_12_long[2:1]  * 1 %}
{% set d4  = product_id_12_long[3:1]  * 3 %}
{% set d5  = product_id_12_long[4:1]  * 1 %}
{% set d6  = product_id_12_long[5:1]  * 3 %}
{% set d7  = product_id_12_long[6:1]  * 1 %}
{% set d8  = product_id_12_long[7:1]  * 3 %}
{% set d9  = product_id_12_long[8:1]  * 1 %}
{% set d10 = product_id_12_long[9:1]  * 3 %}
{% set d11 = product_id_12_long[10:1] * 1 %}
{% set d12 = product_id_12_long[11:1] * 3 %}

// Make a total of the multiplied digits
{% set total = d1 + d2 + d3 + d4 + d5 + d6 + d7 + d8 + d9 + d10 + d11 + d12 %}

// Calculate 13th digit
{% set d13 = (10 - total % 10) % 10 %}

// Set GTIN13
{% set GTIN13 = product_id_12_long ~ d13 }

{% endfor %}
```

### Short custom feed example
```
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0" xmlns:g="http://base.google.com/ns/1.0">
    <channel>
        <title>{{ shop.title | cdata }}</title>
        <link>{{ shop.domain }}</link>
        <description>{{ shop.description | cdata }}</description>
        {% for product in products %}
      		{% set reverse_product_id = product.id | reverse ~ "000000000000" %}
      		{% product_id_12_long = reverse_product_id[:12] | reverse %}
      		{% set d1  = product_id_12_long[0:1]  * 1 %}
      		{% set d2  = product_id_12_long[1:1]  * 3 %}
      		{% set d3  = product_id_12_long[2:1]  * 1 %}
      		{% set d4  = product_id_12_long[3:1]  * 3 %}
      		{% set d5  = product_id_12_long[4:1]  * 1 %}
      		{% set d6  = product_id_12_long[5:1]  * 3 %}
      		{% set d7  = product_id_12_long[6:1]  * 1 %}
      		{% set d8  = product_id_12_long[7:1]  * 3 %}
      		{% set d9  = product_id_12_long[8:1]  * 1 %}
      		{% set d10 = product_id_12_long[9:1]  * 3 %}
      		{% set d11 = product_id_12_long[10:1] * 1 %}
      		{% set d12 = product_id_12_long[11:1] * 3 %}
      		{% set total = d1 + d2 + d3 + d4 + d5 + d6 + d7 + d8 + d9 + d10 + d11 + d12 %}
      		{% set d13 = (10 - total % 10) % 10 %}
      		{% set GTIN13 = product_id_12_long ~ d13 }
          
        	<item>
            <g:id>{{ product.id }}</g:id>
            <title>{{ product.fulltitle | cdata }}</title>
				    <g:gtin>{{ GTIN13 | cdata }}</g:gtin>
          </item>
        {% endfor %}
    </channel>
</rss>
```
### Short custom feed output
```
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0" xmlns:g="http://base.google.com/ns/1.0">
    <channel>
        <title>Zero Wing Shop</title>
        <link>zero-wing.shop</link>
        <description><![CDATA[All your products are belong to us]]></description>
        <item>
        	<g:id>12345678</g:id>
            <title>Aeroplane with wings</title>
			<g:gtin>0000123456784</g:gtin>       	
        </item>
        <item>
        	<g:id>23456789</g:id>
            <title>Cat with wings</title>
			<g:gtin>0000234567898</g:gtin>       	
        </item>
        <item>...</item>
        <item>...</item>
    </channel>
</rss>
```

### Full examples
Please check the custom-rss-feed-code.txt ande custom-rss-feed-output.xml files.
