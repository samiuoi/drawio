graph TD
    Client((END USERS))

    %% 1. Separate Internal DNS into its own frame (Centralized AD DNS)
    subgraph Central_DNS [AD DNS]
        IntDNS[AD DNS Server<br/>Internal Resolution & Forwarding<br/>appa.vpbank.com.vn -> <b style='font-size:20px'>CNAME appa.gslb.vpbank.com.vn</b>]
    end

    subgraph Layer_DNS [GSLB DC/DR Onprem]
        CitrixADNS[Citrix ADNS<br/>Resolve <b style='font-size:20px'>appa.gslb.vpbank.com.vn</b>]
        CitrixGSLB{{Citrix GSLB<br/>Active-Active LB}}
    end

    %% Client DNS Flow
    Client -. "1. Access <b style='font-size:22px'>appa.vpbank.com.vn</b>" .-> Central_DNS
    IntDNS -. "2. Query CNAME" .-> CitrixADNS
    CitrixADNS -. "4. Return healthy Endpoint (IP)" .-> IntDNS
    IntDNS -. "5. Return healthy Endpoint (IP)" .-> Client
    CitrixADNS --- CitrixGSLB

    %% ================= AWS =================
    FW_AWS[Firewall]
    NLB_AWS[LB]
    FW_AWS --> NLB_AWS
    NLB_AWS --> SiteAWS

    subgraph SiteAWS [Site AWS]
        direction TB
        
        AppA_AWS[App A]
        AppB_AWS[App B]
        AppC_AWS[App C]
    end

    %% ================= DC =================
    FW_DC[Firewall]
    NLB_DC[LB]
    FW_DC --> NLB_DC
    NLB_DC --> Site1

    subgraph Site1 [Site DC On-prem]
        direction TB

        AppA_DC[App A]
        AppB_DC[App B]
        AppC_DC[App C]
    end


    %% ================= LCP =================
    FW_LCP[Firewall]
    NLB_LCP[LB]
    FW_LCP --> NLB_LCP
    NLB_LCP --> Site3

    subgraph Site3 [Site Local Cloud]
        direction TB

        AppA_LCP[App A]
        AppB_LCP[App B]
        AppC_LCP[App C]
    end

    %% ================= DR =================
    FW_DR[Firewall]
    NLB_DR[LB]
    FW_DR --> NLB_DR
    NLB_DR --> Site2

    subgraph Site2 [Site DR On-prem]
        direction TB
        
        AppA_DR[App A]
        AppB_DR[App B]
        AppC_DR[App C]
    end

    %% Client Request Flow (After resolving to public/Firewall IP)
    Client -. "x" .-> FW_DR
    Client -. "x" .-> FW_AWS
    Client -- "6. HTTPS Request" --> FW_LCP
    Client -- "6. HTTPS Request" --> FW_DC

    %% GSLB Health-check Flow
    CitrixGSLB -. "3. Monitor HTTP<br><b style='color:red'>heathgw.apps.workload.aws.name.vpbank.cloud/app/a</b><br>Error code, eg: 500,503,404,..." .-> FW_AWS
    CitrixGSLB -. "3. Monitor HTTP<br><b style='color:green'>heathgw.apps.workload.dc.vpbank.vcp/app/a</b><br>Success code, eg: 200" .-> FW_DC
    CitrixGSLB -. "3. Monitor HTTP<br><b style='color:red'>heathgw.apps.workload.dr.vpbank.vcp/app/a</b><br>Error code, eg: 500,503,404,..." .-> FW_DR
    CitrixGSLB -. "3. Monitor HTTP<br><b style='color:green'>heathgw.apps.workload.lcp.vpbank.vcp/app/a</b><br>Success code, eg: 200" .-> FW_LCP

    %% Keep old Style
    classDef client fill:#e1f5fe,stroke:#03a9f4,stroke-width:2px;
    classDef citrix fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef ocp fill:#e8f5e9,stroke:#4caf50,stroke-width:2px;
    classDef ocp2 fill:#ebf5ff,stroke:#327da8,stroke-width:2px;
    classDef component fill:#ffffff,stroke:#333,stroke-width:1px;
    classDef pod fill:#d1c4e9,stroke:#673ab7,stroke-width:1px;
    classDef textscale font-weight:bold,font-size:20px;

    class Client client;
    class Central_DNS,Client,Layer_DNS textscale;
    class Layer_DNS,CitrixADNS,CitrixGSLB,Central_DNS,IntDNS citrix;
    class Site1,Site2 ocp;
    class Site3,SiteAWS ocp2;
    class Routers_DC,Routers_DR,Routers_LCP,Routers_AWS,LocalDNS_DC,LocalDNS_DR,LocalDNS_LCP,Route53_AWS,FW_DC,FW_DR,FW_LCP,FW_AWS component;
    
    %% Assign App blocks to pod class
    class AppA_DC,AppB_DC,AppC_DC,AppA_DR,AppB_DR,AppC_DR,AppA_LCP,AppB_LCP,AppC_LCP,AppA_AWS,AppB_AWS,AppC_AWS pod;