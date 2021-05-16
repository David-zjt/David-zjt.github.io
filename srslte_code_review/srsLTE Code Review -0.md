# srsLTE Code Review -0

## file tree

**/cmake** is used for cmake tool

**/debian** ***maybe*** in order to adapt the system

**/lib** provide the basic encode/decode functions 

**/srsenb** is a basic lte enb && in the future nr

**/srsepc** is the core network implementation by software

**/srsue** is a lte-ue implementation by the SDR

## srsue

To have the C&CXX program running properly  srslte make split dir of header file and source file. Test dir is the test program of the every module in the srsue.

**srsue** is divided in two main parts: physical layer and the protocol stack. Known that physical layer is cared now in my plan, so we do not read it deeply.

### srsue::stack

The main protocol of the ue(User Equipment) was divided  with three layers : "mac"(media access control layer),"rrc"(radio resource control),and "upper". Because in the lte or nr(5G), upper layer is very flexible, but with the basic mobile communication network partition, mac and rrc are the basic module with no much huge change.

## srsepc

Also have two folder. One with header files and the other  with source file. 

In core-network module, we have "hss" to save and deal with some infomation with home users. "Mbms-gw" is a gateway module.Mme is the most common module to deal with the information from the enodeb. "spgw" is also a gateway.

## srsenb

In the lte communication system, enodeb is a integrated base station system. It has function of both BTS and BSC. 

Due to it is almost the direct communication module to the ue by SDR(Software Defined Radio). so its architecture is almost just like srsue.