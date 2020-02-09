# Integration mellem Woocommerce webshop og E-conomic

## Introduktion

Denne beskrivelse forsøger at specificere integrationen mellem en Dybdahl Erhvervstøj Woocommerce Webshop (Herefter webshop) og E-conomic, hvor formålet er at oprette ordre i E-conomic som blev modtaget i webshoppen.

Definationen af webshop ændre sig en smule igennem beskrivelsen, da der er tale om flere webshops. For nu, bliver der taget udgangspunkt i én webshop, da system beskrivelsen er den samme.

### System beskrivelse
Integrationen opretter identitiske ordrer imellem webshop og E-conomic. Når en ordre er gået igennem fra webshoppen, skal integrationen tilføje en ordre i E-conomic med tilsvarende oplysninger fra webshopordren.

## De foreskellige webshops
Der er tale om 3 forskellige integrationer med hver deres funktionalitet men med samme formål. De tre webshops er

1. Dansk B2C Webshop (Herefter B2C webshop)
2. Dansk B2B Webshop (Herefter B2B webshop)
3. Svensk B2B Webshop (Herefter Norisol webshop)

De tre forskellige integrationer er beskrevet hver for sig med hver deres specifitioner både fra webshoppens såvel som E-conomics side af integrationen.

### Terminologi og antagelser
**Skriv dette færdigt!!!!!!!!!!!!!!!!!!!!!!!!!!**
* `var` er en variable.
* Alle variabelnavne i denne beskrivelse er eksempler.
* Eksempler på ordre fra webshoppen er vist med python dictionary struktur og er **kun** eksempler.

## B2C webshop
Fra en webshop ordre `order` skal der bruges følgende felter

| Felt                     | Beskrivelse                    |
|--------------------------|--------------------------------|
| `shop_type`              | Shoppen som ordren stammer fra |
| `order_number`           | Ordre nummer                   |
| `first_name`             | Købers fornavn                 |
| `last_name`              | Købers efternavn               |
| `mail_adress`            | Købers mail                    |
| `phone_number`           | Købers telefon nr.             |
| `street_name_and_number` | Købers adresse og hus nr.      |
| `zipcode`                | Købers postnr.                 |
| `city`                   | Købers by                      |
| `country`                | Købers land                    |
| `products`               | Liste af bestilte produkter    |
| `payment_method`         | Betalingsmetoden               |
| `shipping_type`          | Forsendelses typen             |
| `shipping_price`         | Forsendelses gebyr             |
| `order_notes`            | Købers bemærkninger til ordren |

Fra `order.products` skal der bruges følgende felter

| Felt            | Beskrivelse                         |
|-----------------|-------------------------------------|
| `sku`           | Varenummer på 18 tegn               |
| `amount`        | Antal af denne vare                 |
| `price`         | Prisen ekskl. moms                  |
| `type`          | Produktets type (Tilknytning)       |
| `custom_fields` | Felter som navnetryk, afdeling osv. |

Herfra vil der blive beskrevet hvad der skal ske i E-conomic når en ordre modtages

### 1. Lav ny ordre
Fra **Salg --> Ny ordre**

### 2. Vælg kunde
*Om dette skal ske WooCommerce-side eller integration-side er for implementeringsspecifikt for os*

Produkter kan have forskellige typer på webshoppen, som er vigtig information for ordren. Hvis `product.type` på bare ét af produkterne er forskellig fra `'Dybdahl Erhvervstøj'` er produktet køb via et foreningslogin på B2C shoppen. Alle produkter med `product.type == 'Dybdahl Erhvervstøj'` er købt af almindelige kunder uden login. (Foreninger kan også have en ordre med blandede produkter fra både deres forening og `'Dybdahl Erhvervstøj'`).

Proceduren for at vælge hvilken kunde som ordren er bestilt af, er som følger:
```
foreach product in order.products
    if product.type is not equal to 'Dybdahl Erhvervtøj'
        return product.type
return 'Dybdahl Erhvervstøj'
```

Herefter kan vi vælge en kunde til ordren i E-conomic:
1. Hvis `'Dybdahl Erhvervstøj'` blev returneret, vælg kunden **B2C - Webshop** med **nr. 2** i E-conomic.
2. Hvis noget forskelligt fra `'Dybdahl Erhvervstøj'` blev returneret, vælg da også **B2C - Webshop** med **nr. 2** i E-conomic men ændre adressefeltet på kunden til den returnerede værdi fra proceduren.

### 3. Oprettelse af felter på ordren
Fælgende overskrifter i dette afsnit stemmer overens med hvad E-conomic kalder dem i deres brugergrænseflade.

#### Betingelser
Hvis `order.payment_method` er betalingskort vælges i E-conomic **Betalingskort** med **nr. 2**
Hvis `order.payment_method` er MobilePay vælges i E-conomic **MobilePay** med **nr. 3** 

#### Noter og referencer
1. Overskrift skal være `'B2C Webshop ' + order.order_number`
2. Tekst 1 skal være `order.mail_address + ' (' + order.phone_number + ')' + \n + order.order_notes`
3. Tekst 2 skal være tomt

#### Levering
Opret nyt leveringssted og:
1. Adresse skal være `order.first_name + ' ' + order.last_name + \n + order.street_name_and_number`
2. Post nr. skal være `order.zipcode`
3. By skal være `order.city`
4. Land skal være `order.country`

Leveringsbetingelser skal være enten GLS eller Afhentning i butik afhængig af `order.shipping_type`

### 4. Oprettelse af ordrelinjer
#### Ordrelinje
1. Indsæt `order.product.sku` til **Varenr.** feltet
2. **Varenavn** skal være det som E-conomic foreslår men `... += order.product.custom_fields`
2. Hvis `order.type` er forskellig fra `'Dybdahl Erhvervstøj'` indsæt da `... += ' - ' + order.type` til varens **Varenavn**
3. Indsæt antal `order.product.amount`
4. Ret **Pris** til `order.product.price` (Da denne kan være forskellig fra E-conomic varens). *Alle priser i E-conomic er uden moms*

Gentag 1, 2, 3, 4 indtil ikke flere produkter.

Den sidste vare der (altid) skal tilføjes er et produkt med e-conomic **Vare nr.: Fragtwebshop**. Prisen ændres til `order.shipping_price`

Hvis der mødes et produkt, som ikke findes i E-conomic, skal der ændres i overskiften på **Noter og referencer**. Der skal tilføjes strengen `' - FEJL I LINJER'` til overskriften. Det er meget vigtigt at integrationen ikke selv prøver at oprette produkter som ikke findes. (Hvis fejlen sker efter det `i`'th produkt, er det lige meget om de `i - 1` produkter forbliver på ordren eller ej)



## B2B webshop
Fra en webshop ordre `order` skal der bruges følgende felter

| Felt                     | Beskrivelse                    |
|--------------------------|--------------------------------|
| `shop_type`              | Shoppen som ordren stammer fra |
| `order_number`           | Ordre nummer                   |
| `first_name`             | Købers fornavn                 |
| `last_name`              | Købers efternavn               |
| `shipping_info`          | Købers leveringssted           |
| `products`               | Liste af bestilte produkter    |
| `shipping_price`         | Forsendelses gebyr             |
| `order_notes`            | Købers bemærkninger til ordren |

Fra `order.products` skal der bruges følgende felter

| Felt            | Beskrivelse                         |
|-----------------|-------------------------------------|
| `sku`           | Varenummer på 18 tegn               |
| `amount`        | Antal af denne vare                 |
| `custom_fields` | Felter som navnetryk, afdeling osv. |

Fra `order.shipping_info` skal der bruges følgende felter

| Felt               | Woocommerce beskrivelse                       | E-conomic beskrivelse |
|--------------------|-----------------------------------------------|-----------------------|
| `customer_id`      | `ID` fra brugere -> groups -> Tilføj adresser | Kunde nr.             |
| `leveringssted_id` | Antal af denne vare                           | Leveringssted nr.     |

Herfra vil der blive beskrevet hvad der skal ske i E-conomic når en ordre modtages

### 1. Lav ny ordre
Fra **Salg --> Ny ordre**

### 2. Vælg kunde

