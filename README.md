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
| `products`               | List af bestilte produkter     |
| `payment_method`         | Betalingsmetoden               |

Fra `products` skal der bruges følgende felter

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
Produkter kan have forskellige typer på webshoppen, som er vigtig information for ordren. Hvis `product.type` er forskellig fra `'Dybdahl Erhvervstøj'` er produktet køb via et foreningslogin på B2C shoppen. Alle produkter med `product.type == 'Dybdahl Erhvervstøj'` er købt af almindelige kunder uden login.

Proceduren for at vælge hvilken kunde som ordren er bestilt af, er som følger:
```
for product in order.products
    if product.type != 'Dybdahl Erhvervtøj'
        return product.type
return 'Dybdahl Erhvervstøj'
```

Herefter kan vi vælge en kunde til ordren i E-conomic:
1. Hvis `'Dybdahl Erhvervstøj'` blev returneret, vælg kunden **B2C - Webshop** med **nr. 2** i E-conomic.
2. Hvis noget forskelligt fra `'Dybdahl Erhvervstøj'` blev returneret, vælg da også **B2C - Webshop** med **nr. 2** i E-conomic men ændre adressefeltet på kunden til den returnerede værdi fra proceduren.

### 3. Oprettelse af felter på ordren

