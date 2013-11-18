fisk.py - simple fiscalization (Fiskalizacija) library 
		  (Hrvatska) 
		  
## REQUIREMENTS

1. pyXMLsec library - http://pyxmlsec.labs.libre-entreprise.org/
2. pyCrypto library - https://www.dlitz.net/software/pycrypto/

## USAGE

### Echo Request

```Python
import fisk
import xml.etree.ElementTree as et

#test echo 
echo = fisk.EchoRequest("Proba echo poruke")

#send request and print server reply
echo_reply = echo.execute()
if(echo_reply != False):
    print echo_reply
else:
    errors = echo.get_last_error()
    print "EchoRequest errors:"
    for error in errors:
        print error
```

### PoslovniProstor Request

```Python
import fisk
import xml.etree.ElementTree as et
from datetime import date, timedelta

#create object needed for signing and verifing xml messages 
signer = fisk.FiskXMLsec('/path/to/your/key.pem', "kaypassword", '/path/to/your/cert.pem', ['/path/to/porezna/rootcert/democacert.pem'])
#create addres
adresa = fisk.Adresa(data = {"Ulica": "Proba", "KucniBroj": "1", "BrojPoste": "54321"})
#create poslovni prostor      
pp = fisk.PoslovniProstor(data = {"Oib": "12345678901",
                             "OznPoslProstora": "POS1",
                             "AdresniPodatak": fisk.AdresniPodatak(adresa),
                             "RadnoVrijeme": "PON-PET 9:00-17:00", 
                             "DatumPocetkaPrimjene": (date.today() + timedelta(days = 1)).strftime('%d.%m.%Y')})

#you can also access (set and get) attributes of fisk element classes as
pp.SpecNamj = "12345678901"
print pp.OznPoslProstora

#poslovni prostor request
ppz = fisk.PoslovniProstorZahtjev(pp)

ppz_reply = ppz.execute(signer)
if(ppz_reply == True):
    print "PoslovniProstorZahtjev seccessfuly sent!"
else:
    errors = ppz.get_last_error()
    print "PoslovniProstorZahtjev reply errors:"
    for error in errors:
        print error
```

## Racun Request

```Python
import fisk
import xml.etree.ElementTree as et
from datetime import date, timedelta

#create object needed for signing and verifing xml messages 
signer = fisk.FiskXMLsec('/path/to/your/key.pem', "kaypassword", '/path/to/your/cert.pem', ['/path/to/porezna/rootcert/democacert.pem'])

racun = fisk.Racun(data = {"Oib": "12345678901",
              "USustPdv": "true",
              "DatVrijeme": "26.10.2013T23:50:00",
              "BrRac": fisk.BrRac({"BrOznRac": "2", "OznPosPr":"POS2", "OznNapUr":"1"}),
              "Pdv": [fisk.Porez({"Stopa":"25.00", "Osnovica":"100.00", "Iznos":"25.00"}), fisk.Porez({"Stopa":"10.00", "Osnovica":"100.00", "Iznos":"10.00"})],
              "Pnp": [fisk.Porez({"Stopa":"25.00", "Osnovica":"100.00", "Iznos":"25.00"}), fisk.Porez({"Stopa":"10.00", "Osnovica":"100.00", "Iznos":"10.00"})],
              "OstaliPor": [fisk.OstPorez({"Naziv": "Neki porez",  "Stopa":"3.00", "Osnovica":"100.00", "Iznos":"3.00"})],
              "IznosOslobPdv": "100.00",
              "IznosMarza": "100.00",
              "IznosNePodlOpor": "50.00",
              "Naknade": [fisk.Naknada({"NazivN" : "test", "IznosN": "10.00"})],
              "IznosUkupno": "500.00",
              "NacinPlac": "G",
              "OibOper": "12345678901",
              "NakDost": "false",
              "ParagonBrRac": "123-234-12",
              "SpecNamj": "Tekst specijalne namjne"},
              keyFileName = '/path/to/your/key.pem')

#IWe did not supplied required element in constructor so now we set it
racun.OznSlijed = "P"

#Zastitni kod is calculated so print it
print "ZKI: " + racun.ZastKod

#change one variable and check new zastitni kod
racun.IznosUkupno = "1233.00"
print "ZKI :" + racun.ZastKod

#create Request and send it to server (DEMO) and print reply
racunZahtjev = fisk.RacunZahtjev(racun)
racun_reply = racunZahtjev.execute(signer)
if(racun_reply != False):
    print "JIR is :" + racun_reply
else:
    errors = racunZahtjev.get_last_error()
    print "RacunZahtjev reply errors:"
    for error in errors:
        print error
```

## KEY GENERATION

Fiscalization keys and certificates are delivered in .p12 or .pfx format. To be used with this library you should
convert them to .pem format. This can be done with openssl.

```
openssl pkcs12 -in certificate.pfx -out certificate.pem -nodes
```

Now certificate.pem holds both key and certificate. So you should manually open this file and copy each to
separate file including BEGIN/END statements.

### CA Certificates
You will also need CA certificate for DEMO and PRODUCTION environment. This certificate is needed for
verification process.

#### DEMO CA Certificate

You can download this certificate https://demo-pki.fina.hr/crl/democacert.cer

#### PRODUCTION CA Certificate

You can found it in .p12 in which you have received your private key and certificate.

## Changelog

### Version 0.6
  * Known bugs fixed
  * added execute method to FiskXMLRequest for easier request handling (no more need to parse raw XML returned
  from server and better error handling). execute method is similar to send method but it parses reply and
  returns reply as str or False if error occurs. There is also new get_last_error method for accessing
  possible errors.
  * USAGE section in README now have examples using execute method instead send method. Send method still works
  as it is used internally
  * little better handling of connection errors of FiskSOAPClient

### Version 0.5.1

  * Known bugs fixed
  * required element check moved to generate method. So required element is check before sending request
  not at constructor. 

### Version 0.5

First public release 