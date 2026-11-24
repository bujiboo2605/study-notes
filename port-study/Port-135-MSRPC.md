# Port 135: Microsoft RPC (MSRPC)

> **Study notes:** Networking / Windows / Ports and Protocols\
> **Port:** TCP 135\
> **Service label:** MSRPC (Microsoft Remote Procedure Call)\
> **Purpose of these notes:** Understand what MSRPC does, how RPC
> communication works, how to identify exposed RPC interfaces, and why
> they matter to both penetration testers and defenders.

------------------------------------------------------------------------

## 1. Introduction

### 1.1 My initial understanding

MSRPC is Microsoft's implementation of Remote Procedure Call (RPC). It
allows a program running on one computer (Computer A) to request that
another computer (Computer B) perform a specific operation exposed by
one of its services.

For example, an application on my PC could request an operation from a
service running on my home server or another laptop.

### 1.2 Refined understanding

RPC does more than simply let one program access a function on another
computer. It provides the communication mechanism that allows the client
and server programs to exchange procedure calls and responses. The RPC
runtime and transport mechanisms handle much of the communication work
so that the application does not have to implement all the networking
details itself.

A useful definition from HackTricks:

> Microsoft Remote Procedure Call (MSRPC) is a client-server mechanism
> that lets a program invoke a procedure exposed by another process,
> locally or over a network, through generated stubs and
> runtime-selected transports.

### 1.3 Important terminology

  -----------------------------------------------------------------------
  Technical term                      Simple explanation
  ----------------------------------- -----------------------------------
  Client-server mechanism             One program requests an operation
                                      and another program provides it.

  Invoke a procedure                  Ask a program to execute a function
                                      or operation.

  Exposed by another process          A program makes certain functions
                                      available for other programs to
                                      use.

  Locally or over a network           The communicating programs can be
                                      on the same computer or on
                                      different computers.

  Generated stubs                     Helper code that packages requests
                                      and interprets responses so the
                                      application does not need to handle
                                      every communication detail
                                      manually.

  Runtime-selected transport          The RPC runtime uses an
                                      available/configured transport to
                                      carry the request.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 2. How MSRPC works

The RPC communication process involves two important pieces:

1.  **RPC stubs**
2.  **The RPC runtime**, commonly provided on Windows by `rpcrt4.dll`

### 2.1 RPC stubs

RPC stubs are generated helper code associated with the client and
server applications.

The client-side stub helps prepare a procedure call for transmission.
The server-side stub helps interpret the incoming request and pass the
operation to the server application. The stubs also help package
parameters and interpret the response.

In practical terms, think of a stub as a translator between the
application and the RPC runtime.

The client and server have corresponding stub code for the RPC
interface. The stubs know the procedure's expected parameters and data
types, and help marshal and unmarshal the data.

**Marshalling** means converting parameters into a representation
suitable for transmission.

**Unmarshalling** means interpreting the received representation and
reconstructing the values.

The stubs are generated from an RPC interface definition, commonly
written using an Interface Definition Language (IDL). The IDL describes
the interface, including the operations and their parameters. A compiler
can use that definition to generate the stub code.

### 2.2 RPC runtime: `rpcrt4.dll`

`rpcrt4.dll` is a Windows RPC runtime library.

The RPC runtime provides the functions and mechanisms used by the client
and server stubs. It helps manage RPC calls, data representation,
bindings, and communication over the selected transport.

A simplified analogy: the stubs prepare and interpret the message, while
the runtime helps carry out the RPC communication.

### 2.3 How stubs communicate with the runtime

The stubs do not independently establish the entire network
conversation. They call the RPC runtime APIs in their own process. The
runtime handles the RPC communication using the configured transport.

A simplified sequence:

1.  The client application calls its generated client stub.
2.  The client stub prepares the procedure parameters.
3.  The stub calls the appropriate RPC runtime API.
4.  The RPC runtime processes the request and uses the configured
    transport.
5.  The operating system's network stack sends the data if the server is
    remote.
6.  The server-side runtime and stub receive and interpret the request.
7.  The server application performs the requested operation.
8.  The response travels back through the server and client RPC
    components.

The exact details depend on the RPC interface, binding, transport, and
implementation.

### 2.4 Network Data Representation (NDR)

RPC parameters need a consistent representation when they are exchanged
between systems. Microsoft's RPC commonly uses Network Data
Representation (NDR) to marshal data.

NDR defines how data types and values are represented for RPC
communication, helping systems with different internal representations
exchange information.

### 2.5 Transports

RPC can use different transports. The diagram in the original notes
(Figure 1.1, page 2) illustrates RPC communication through transports
such as TCP/IP, named pipes over SMB, and HTTP-related mechanisms.

Examples include:

-   RPC over TCP/IP.
-   RPC over named pipes, commonly carried over SMB.
-   RPC over HTTP in supported configurations.

**Important:** TCP port 135 is associated with the RPC Endpoint Mapper.
It is not the only port used by RPC. A client may contact the Endpoint
Mapper and then connect to a separate endpoint where the requested RPC
interface is available.

------------------------------------------------------------------------

## 3. Identifying exposed RPC services

A penetration tester may want to identify which RPC interfaces are
exposed and how they can be reached.

RPC enumeration can involve querying the Endpoint Mapper and examining
individual RPC endpoints. Tools such as `rpcdump.py` can display
interface identifiers and binding information.

### 3.1 RPC interface identifiers

An RPC interface is identified by a UUID, also called an interface
identifier (IFID).

Enumeration output may include:

  -----------------------------------------------------------------------
  Field                               Meaning
  ----------------------------------- -----------------------------------
  IFID / UUID                         Identifies the RPC interface.

  Version                             Indicates the interface version.

  Annotation                          A descriptive label, when
                                      available.

  Binding                             Describes how the interface can be
                                      reached, including the transport
                                      and endpoint details.
  -----------------------------------------------------------------------

For example, a binding may look like:

``` text
ncacn_ip_tcp:10.10.10.5[49664]
```

This indicates an RPC interface reachable over TCP at the specified host
and endpoint port.

The original notes include an example `rpcdump` output in Figure 1.2
(page 3), showing an interface identifier, version, annotation, UUID and
binding.

### 3.2 Questions to answer during RPC enumeration

When investigating RPC, consider:

-   Which RPC interfaces are accessible?
-   Which endpoints are associated with registered or accessible
    interfaces?
-   Are there interfaces that ordinary endpoint enumeration does not
    reveal?
-   Does an accessible service expose management information?
-   Are there signs of a security weakness that warrant further
    investigation?

Enumeration results identify possible functionality. They do not, by
themselves, prove that a service is vulnerable or that an operation can
be performed without authorization.

------------------------------------------------------------------------

## 4. RPC enumeration tools

The original notes identify the following Metasploit auxiliary scanner
modules:

``` text
auxiliary/scanner/dcerpc/endpoint_mapper
auxiliary/scanner/dcerpc/hidden
auxiliary/scanner/dcerpc/management
auxiliary/scanner/dcerpc/tcp_dcerpc_auditor
```

Use these only on systems you own or are explicitly authorized to
assess.

### 4.1 Endpoint Mapper

**Purpose:** Enumerates RPC endpoints and discovers information about
registered RPC interfaces and their bindings.

Example of the kind of information an endpoint listing may show:

``` text
Interface UUID: 12345678-...
Binding: ncacn_ip_tcp:10.10.10.5[49664]
```

This indicates that an RPC interface is associated with a TCP endpoint
on port `49664`. It gives the tester another service endpoint to
investigate, rather than only knowing that TCP 135 is open.

### 4.2 Hidden interface enumeration

**Purpose:** Attempts to identify RPC interfaces that may not appear in
ordinary Endpoint Mapper listings by using additional RPC management
functionality.

An interface not appearing in a basic endpoint listing should not
automatically be considered exploitable. Any discovered interface
requires further validation.

### 4.3 Management interface enumeration

**Purpose:** Queries RPC management functionality for information about
accessible RPC servers and interfaces.

This may help identify RPC functionality exposed through management
operations.

### 4.4 TCP DCE/RPC auditor

**Purpose:** Examines TCP services for DCE/RPC.

It is not limited to querying the Endpoint Mapper on port 135. This
matters because an RPC service may listen on a separate TCP endpoint
after the Endpoint Mapper has helped a client discover it.

### 4.5 `rpcdump.py`

`rpcdump.py` can display information about RPC endpoints.

Important output fields:

-   **IFID / UUID:** Identifies the RPC interface.
-   **Version:** Interface version.
-   **Annotation:** A descriptive label, if available.
-   **Binding:** Shows how the interface can be reached, including
    transport and endpoint details.

------------------------------------------------------------------------

## 5. Notable MSRPC interfaces

The following interfaces were identified in the original study notes,
with information attributed there to HackTricks. Their availability and
the operations permitted depend on the Windows version, configuration,
transport, authentication and account permissions.

### 5.1 Local Security Authority (LSA)

  Field        Information
  ------------ -----------------------------------------------
  IFID         `12345778-1234-abcd-ef00-0123456789ab`
  Named pipe   `\pipe\lsarpc`
  Purpose      Queries Local Security Authority information.

Possible information includes users, Security Identifiers (SIDs), and
security policies, depending on permissions.

**Security relevance:** May help identify security-related information
and understand aspects of the target's security configuration.

### 5.2 LSA Directory Services (LSA-DS)

  Field        Information
  ------------ -------------------------------------------------------
  IFID         `3919286a-b10c-11d0-9ba8-00c04fd92ef5`
  Named pipe   `\pipe\lsarpc`
  Purpose      Provides access to LSA Directory Services operations.

Can provide information about domains and domain trust relationships,
subject to permissions.

**Security relevance:** Domain and trust information can help map
relationships in an Active Directory environment.

### 5.3 Security Account Manager Remote Protocol (SAMR)

  ----------------------------------------------------------------------------
  Field                               Information
  ----------------------------------- ----------------------------------------
  IFID                                `12345778-1234-abcd-ef00-0123456789ac`

  Named pipe                          `\pipe\samr`

  Purpose                             Supports operations involving local or
                                      domain accounts, groups and aliases.
  ----------------------------------------------------------------------------

Depending on permissions and configuration, SAMR operations may expose
account and group information, and potentially password-policy
information.

**Security relevance:** Helps identify user accounts, group membership
and potentially privileged accounts.

**Important:** Password guessing through SAMR can trigger account
lockout policies. SAMR does not bypass these protections.

### 5.4 AT-Scheduler (Task Scheduler interface)

  Field        Information
  ------------ --------------------------------------------------------
  IFID         `1ff70682-0a51-30e8-076d-740be8cee98b`
  Named pipe   `\pipe\atsvc`
  Purpose      Associated with the legacy AT-Scheduler RPC interface.

Supports remote scheduled-task operations on compatible Windows systems,
when the account has sufficient permissions.

**Security relevance:** Misconfigured permissions may allow unauthorized
task creation or execution.

### 5.5 Remote Registry

  Field        Information
  ------------ ----------------------------------------------------------
  IFID         `338cd001-2244-31f1-aaaa-900038001003`
  Named pipe   `\pipe\winreg`
  Purpose      Provides remote access to parts of the Windows Registry.

Depending on permissions, clients may query or modify registry keys and
values.

**Security relevance:** Excessive permissions may expose sensitive
configuration data or allow unauthorized registry modifications.

### 5.6 Service Control Manager (SCM)

  Field        Information
  ------------ --------------------------------------------------------
  IFID         `367abb81-9844-35f1-ad32-98f038001003`
  Named pipe   `\pipe\svcctl`
  Purpose      Provides access to Service Control Manager operations.

Can support service enumeration, querying service status or
configuration, and service operations such as starting and stopping
services, subject to permissions.

**Security relevance:** Excessive service-control permissions may allow
unauthorized service modifications or contribute to privilege
escalation.

**Important distinction:** SCM is responsible for service control, not
general server-share enumeration. Remote command execution is not
automatic; it depends on permissions and service configuration.

### 5.7 Server Service (SRVSVC)

  ----------------------------------------------------------------------------
  Field                               Information
  ----------------------------------- ----------------------------------------
  IFID                                `4b324fc8-1670-01d3-1278-5a47bf6ee188`

  Named pipe                          `\pipe\srvsvc`

  Purpose                             Provides information about server
                                      resources and SMB-related services.
  ----------------------------------------------------------------------------

Can support share enumeration and queries about server information.
Depending on supported operations and permissions, it may also provide
information about sessions and connections.

**Security relevance:** Share enumeration helps identify accessible
folders and resources that may require further security assessment.

**Important distinction:** SRVSVC handles Server Service operations.
Starting and stopping Windows services is associated with SCM/SVCCTL.

### 5.8 RPC Endpoint Mapper

  ----------------------------------------------------------------------------
  Field                               Information
  ----------------------------------- ----------------------------------------
  IFID                                `4d9f4ab8-7d1c-11cf-861e-0020af6e7c57`

  Named pipe                          `\pipe\epmapper`

  Purpose                             Helps clients discover registered RPC
                                      interfaces and their communication
                                      endpoints.
  ----------------------------------------------------------------------------

Maps RPC interfaces to the endpoints where they can be reached and
supports RPC service discovery and connection establishment.

**Security relevance:** Endpoint enumeration may reveal additional RPC
services and interfaces for further investigation.

**Important:** The Endpoint Mapper is used for service discovery, not
password brute-forcing.

------------------------------------------------------------------------

## 6. RPC interface summary

  RPC interface     Named pipe         Main purpose
  ----------------- ------------------ -----------------------------------------
  LSA               `\pipe\lsarpc`     Security information and policy queries
  LSA-DS            `\pipe\lsarpc`     Domain and trust information
  SAMR              `\pipe\samr`       User, group and account operations
  AT-Scheduler      `\pipe\atsvc`      Legacy scheduled-task operations
  Remote Registry   `\pipe\winreg`     Remote Registry access
  SCM / SVCCTL      `\pipe\svcctl`     Service enumeration and control
  SRVSVC            `\pipe\srvsvc`     Server and share information
  Endpoint Mapper   `\pipe\epmapper`   RPC endpoint discovery

------------------------------------------------------------------------

## 7. Red Team perspective

When assessing MSRPC in an authorized environment, the goal is to
understand which interfaces are exposed and whether their operations
reveal information or permit actions beyond what should be allowed.

Areas to investigate:

-   Accessible RPC interfaces and their bindings.
-   User, group and domain information exposed through permitted
    operations.
-   Service configuration and permissions.
-   Remote Registry access and permissions.
-   Server share and session information.
-   Whether management operations are exposed unnecessarily.
-   Whether access controls are correctly enforced.

An exposed RPC interface is an attack surface, not proof of a
vulnerability. Validate the permissions and behavior of each relevant
operation.

## 8. Blue Team perspective

Defenders should understand which RPC services are required for business
operations and restrict unnecessary exposure.

Areas to investigate:

-   Unexpected RPC connections between workstations and servers.
-   Unusual access to administrative named pipes.
-   Unexpected remote service-control activity.
-   Unusual remote Registry access.
-   Unexpected account or group enumeration.
-   RPC traffic from systems that should not administer the target.
-   Firewall rules and dynamic RPC endpoint exposure.

Useful evidence may come from Windows security logs, endpoint telemetry,
firewall logs and network traffic analysis. The exact event sources and
event identifiers depend on the activity and audit configuration.

## 9. Key takeaways

-   MSRPC allows programs to invoke operations exposed by other
    processes, locally or over a network.
-   RPC stubs help marshal requests and unmarshal responses.
-   `rpcrt4.dll` provides Windows RPC runtime functionality.
-   RPC can use different transports, including TCP/IP and named pipes
    over SMB.
-   TCP 135 is associated with the RPC Endpoint Mapper, which helps
    clients discover RPC endpoints.
-   RPC interfaces can be identified by UUIDs/IFIDs.
-   RPC enumeration can reveal interfaces, bindings and additional
    endpoints.
-   The presence of an interface does not prove that its operations are
    accessible or vulnerable.
-   Authentication, authorization, configuration and permissions
    determine what a remote client can actually do.

## 10. References and further reading

### Sources mentioned in the original notes

-   HackTricks: MSRPC --- https://book.hacktricks.wiki/
-   Microsoft Open Specifications: MS-RPCE ---
    https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rpce/
-   Microsoft RPC troubleshooting ---
    https://learn.microsoft.com/en-us/troubleshoot/windows-client/networking/rpc-errors-troubleshooting

### Tools mentioned

-   Metasploit Framework --- https://docs.metasploit.com/
-   Impacket --- https://github.com/fortra/impacket
-   Wireshark --- https://www.wireshark.org/docs/

------------------------------------------------------------------------

## 11. Questions to investigate next

Use this section to record questions that remain unanswered during your
study.

-   [ ] How does an RPC interface definition (IDL) describe procedures
    and parameters?
-   [ ] How does NDR represent complex data types?
-   [ ] What is an RPC binding, and how is it created?
-   [ ] What is the difference between the Endpoint Mapper and an RPC
    server endpoint?
-   [ ] How does RPC over named pipes differ from RPC over TCP?
-   [ ] Which Windows event sources can help investigate remote RPC
    activity?
-   [ ] How can Wireshark identify DCE/RPC traffic?
-   [ ] Which RPC interfaces are commonly exposed in a Windows Active
    Directory environment?

------------------------------------------------------------------------

*Personal study notes. Review the protocol specification and tool
documentation before relying on implementation details or interface
identifiers during an assessment.*
