# Software Architecture

Figure 1.1 shows the software architecture of the TCC8050. The three cores consist of the following features, each of which exchanges information over the IPC.

The TCC8050 uses the CA72 (main core) to provide multimedia functionality, and the CA53 (sub-core) provides immediate processing of the main core during operation (for example, rear camera). The CR5 provides a real-time OS to provide the capabilities used to communicate with a car.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/161264431/7a6c7f14-8f83-4976-a6bc-3f78a4d576b3">
</p>
<p align="center"><strong>Figure 1.1 TCC8050 Software Architecture</strong></p>
