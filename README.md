# pypassport-2.0
A clone of the [pypassport-2.0 repo](https://github.com/landgenoot/pypassport-2.0).

## Changes I made

Changing protocol from T0 to T1, based on experiments done using [gscriptor](ludovic.rousseau.free.fr/softwares/pcsc-tools/)
```
# self._pcsc_connection.connect(self.sc.scard.SCARD_PCI_T0)
self._pcsc_connection.connect(self.sc.scard.SCARD_PCI_T1)
```
pypassport > reader.py > class PcscReader > def connect: 

Adding this line,
```
mrz = self._mrz
```
to pypassport > doc9303 > mrz.py > class MRZ > def _checkDigitsTD1 & def _checkDigitsTD2

When running the 'EPassport.readPassport' method we came across an error with this message:
```
('Data not found:', '6F63')
```
This is most likely a merging of the '6F' and '63' which are the locations of DG15 (Public Keys) and DG3 (Finger Print) respectively on the LDS (Logical Data Structure). For some reason these two tags are stored conjoined in the common file, which contains a list of available DG's. This list can be read using 'EPassport.readCom'.

I added this code to 'readCom', which does not yet fix the whole problem:
```
# Temp fix for 6F63 Data not found issue
double_tag = False
ef_com = self["Common"]["5C"]
for tag in ef_com:
  if tag == "6F63":
  ef_com.append("6F")
  ef_com.append("63")
  double_tag = True
        
  if double_tag:
  ef_com.remove("6F63")
        
 # print(ef_com)
 ###
```

For some reason DG15 returns empty and for DG3 the 'securty status is not satisfied'.
For now this can be skipped, with the info from DG15 (public keys) the info on the NFC can be varified as valid, but this is not relevant for us at this moment. We also don't need DG3 (Finger Print).
