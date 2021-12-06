# HPE-Synergy-Software-Releases-API

This API project provides the ability to perform `get` operations to collect information about HPE Synergy Software Releases.

'db.json' was built on the data collected on December 6, 2021 from:

- https://techhub.hpe.com/eginfolib/synergy/sw_release_info/index.html
- https://techhub.hpe.com/eginfolib/synergy/sw_release_info/OS_Support.html

This API allows anyone to retrieve and use this data programmatically to feed automation engines and codes.

The information supported by the API is as follows:

- HPE Synergy Software Releases accessible from https://hpe-synergy-software-api.herokuapp.com/synergyManagementCombinations
- Operating systems supported by HPE Synergy from https://hpe-synergy-software-api.herokuapp.com/synergyOsSupport

## Ansible: Gather HPE Synergy Management Combination information

```
---
- name: Gather HPE Synergy Management Combination information
  hosts: localhost

  tasks:
    - name: Gather facts about Synergy Management Combinations
      uri:
        url: https://hpe-synergy-software-api.herokuapp.com/synergyManagementCombinations
      register: response
      delegate_to: localhost

    # - debug: var=response.json

    - name: Getting supported Synergy Service Packs for Oneview 6.30/Image Streamer 6.10
      set_fact:
        supported_ssp_for_6_30: "{{ response.json |
          selectattr('composer', 'equalto', '6.30') |
          selectattr('imageStreamer', 'equalto', '6.10') |
          map(attribute='supportedSsp') |
          list }}"

    - debug: var=supported_ssp_for_6_30

```

Output:

```
ansible-playbook get-Synergy-Management-Combination.yml

PLAY [Gather HPE Synergy Management Combination information] *************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Gather facts about Synergy Management Combinations] ****************************************************************************************************************************************************
ok: [localhost -> localhost]

TASK [Getting supported Synergy Service Packs for Oneview 6.30/Image Streamer 6.10] **************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "supported_ssp_for_6_30": [
        [
            "2021.11.01",
            "2021.05.03",
            "2021.05.01",
            "2021.02.02",
            "2021.02.01",
            "2021.01.03",
            "2021.01.02",
            "2021.01.01"
        ]
    ]
}

PLAY RECAP ***************************************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Ansible: Gather HPE Synergy OS Support information

```
---
- name: Gather HPE Synergy OS Support information
  hosts: localhost

  tasks:
    - name: Gather facts about Synergy OS Support
      uri:
        url: https://hpe-synergy-software-api.herokuapp.com/synergyOsSupport
      register: response
      delegate_to: localhost

    # - debug: var=response.json[4]

    - name: Getting supported Synergy Service Packs for VMware ESXi
      set_fact:
        vmwareEsxi: "{{ response.json[4].vmwareEsxi }}"

    #- debug: var=vmwareEsxi

    - set_fact:
        supported_osVersions_for_480Gen10: "{{ vmwareEsxi |
          selectattr('model', 'equalto', 'SY 480 Gen10') |
          map(attribute='osVersions') |
          list }}"
        supported_osVersions_for_480Gen10plus: "{{ vmwareEsxi |
          selectattr('model', 'equalto', 'SY 480 Gen10 Plus') |
          map(attribute='osVersions') |
          list }}"

    #- debug: var=supported_osVersions_for_480

    - name: Getting supported Synergy Service Packs for VMware ESXi 7.0 Update 3 with HPE Synergy 480 Gen10
      set_fact:
        supported_ssp_for_480Gen10_with_ESXi70U3:
          "{{ supported_osVersions_for_480Gen10[0] |
          selectattr('osVersion', 'equalto', 'VMware ESXi 7.0 Update 3') |
          list }}"

    - debug: var=supported_ssp_for_480Gen10_with_ESXi70U3

    - name: Getting supported Synergy Service Packs for VMware ESXi 6.7 Update 3 from May 2021 with HPE Synergy 480 Gen10
      set_fact:
        supported_ssp_for_480Gen10_with_ESXi70U2:
          "{{ supported_osVersions_for_480Gen10[0] |
          selectattr('osVersion', 'equalto', 'VMware ESXi 6.7 Update 3') |
          selectattr('releaseDate', 'equalto', '2021-05-01') |
          list }}"

    - debug: var=supported_ssp_for_480Gen10_with_ESXi70U2

    - name: Getting supported Synergy Service Packs for VMware ESXi 7.0 Update 3 with SHPE Synergy 480 Gen10+
      set_fact:
        supported_ssp_for_480Gen10plus_with_ESXi70U3:
          "{{ supported_osVersions_for_480Gen10plus[0] |
          selectattr('osVersion', 'equalto', 'VMware ESXi 7.0 Update 3') |
          list }}"

    - debug: var=supported_ssp_for_480Gen10plus_with_ESXi70U3


```

Output:

```
ansible-playbook get-Synergy-OS-Support.yml

PLAY [Gather HPE Synergy OS Support information] ************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************
ok: [localhost]

TASK [Gather facts about Synergy OS Support] ****************************************************************************************************************
ok: [localhost -> localhost]

TASK [Getting supported Synergy Service Packs for VMware ESXi] **********************************************************************************************
ok: [localhost]

TASK [set_fact] *********************************************************************************************************************************************
ok: [localhost]

TASK [Getting supported Synergy Service Packs for VMware ESXi 7.0 Update 3 with HPE Synergy 480 Gen10] ******************************************************
ok: [localhost]

TASK [debug] ************************************************************************************************************************************************
ok: [localhost] => {
    "supported_ssp_for_480Gen10_with_ESXi70U3": [
        {
            "osVersion": "VMware ESXi 7.0 Update 3",
            "releaseDate": "2021-11-01",
            "supportedSsp": [
                "2021.11.01"
            ],
            "vlcmSupport": "true"
        }
    ]
}

TASK [Getting supported Synergy Service Packs for VMware ESXi 6.7 Update 3 from May 2021 with HPE Synergy 480 Gen10] ****************************************
ok: [localhost]

TASK [debug] ************************************************************************************************************************************************
ok: [localhost] => {
    "supported_ssp_for_480Gen10_with_ESXi70U2": [
        {
            "osVersion": "VMware ESXi 6.7 Update 3",
            "releaseDate": "2021-05-01",
            "supportedSsp": [
                "2021.11.01",
                "2021.05.03",
                "2021.05.01"
            ],
            "vlcmSupport": "false"
        }
    ]
}

TASK [Getting supported Synergy Service Packs for VMware ESXi 7.0 Update 3 with SHPE Synergy 480 Gen10+] ****************************************************
ok: [localhost]

TASK [debug] ************************************************************************************************************************************************
ok: [localhost] => {
    "supported_ssp_for_480Gen10plus_with_ESXi70U3": [
        {
            "osVersion": "VMware ESXi 7.0 Update 3",
            "releaseDate": "2021-11-01",
            "supportedSsp": [
                "2021.11.01"
            ],
            "vlcmSupport": "true"
        }
    ]
}

PLAY RECAP **************************************************************************************************************************************************
localhost                  : ok=10   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

## PowerShell: Gather HPE Synergy Management Combination information

To get supported HPE Synergy Service Packs for HPE Oneview 6.40 and HPE Image Streamer 6.40:

```
$response = Invoke-RestMethod 'https://hpe-synergy-software-api.herokuapp.com/synergyManagementCombinations' -Method 'GET'

$response | ? { $_.composer -eq "6.40" -and $_.imagestreamer -eq "6.40"} | % supportedssp
```

Output:

```
2021.11.01
2021.05.03
2021.05.01
2021.02.02
2021.02.01
2021.01.03
2021.01.02
2021.01.01

```

## PowerShell: Gather HPE Synergy OS Support information

To get supported Synergy Service Packs for VMware ESXi 7.0 Update 3 with HPE Synergy 480 Gen10

```
$response = Invoke-RestMethod 'https://hpe-synergy-software-api.herokuapp.com/synergyOsSupport' -Method 'GET'

($response[4].vmwareEsxi | ? { $_.model -eq "SY 480 Gen10" }).osVersions | ? osVersion -eq "VMware ESXi 7.0 Update 3"

```

Output:

```
osVersion                releaseDate vlcmSupport supportedSsp
---------                ----------- ----------- ------------
VMware ESXi 7.0 Update 3 2021-11-01  true        {2021.11.01}

```
