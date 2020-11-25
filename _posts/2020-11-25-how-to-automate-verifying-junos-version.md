# fantastic-couscous

```yaml
---
- name: Verify Junos version
  hosts: all
  connection: local
  gather_facts: no
  roles:
  - Juniper.junos
  vars:
    ansible_python_interpreter: "/usr/bin/python2.7"
  tasks:
```

The code above is the start of our ansible-playbook, which defines:

* Execute against all 'junos' hosts in our inventory.yaml file
* Execute using local resources, using the local Python interpreter to save device resources (or, potentially, the device does not have a Python interpreter)
* Do not 'gather facts' about the target connection. This is implicit as the connection is local.
* Import the 'juniper.junos' Ansible role, so that we may call on the juniper.junos.rpc function.

In order to prep our environment we're going to make use of some built-in, core modules in the following two tasks:

```yaml
		- name: Ansible create file if it doesn't exist example
      file:
        path: "{{ inventory_hostname }}_bgp_report.txt"
        state: absent
		- name: Ansible create file if it doesn't exist example
      file:
        path: "{{ inventory_hostname }}_bgp_report.txt"
        state: touch
```

The first task delete's any remnants from previous runs of the playbook, and the second creates a blank file in its place.

After executing the RPC module against a Junos device, the device will reply with an XML document containing the information requested. We will want to save this XML document in a temporary location for later.

To achieve this we add two more tasks that simply destroy and recreate a temporary folder each time the playbook is run, similar to the two tasks above. Notice the difference between the state in the tasks that handle files and those that handle directories.

```
    - name: remove host build temp directory
      file: 
        path: "{{playbook_dir}}/tmp"
        state: absent
    - name: create host build temp directory
      file:
        path: "{{playbook_dir}}/tmp"
        state: directory
```


Now that we have the tasks created to facilitate our environment, we want to actually call the RPC request and collect the data we're looking for. Unfortunately, the best way to identify the RPC needed is to logon to a Junos device and run a command:

` show bgp summary | display xml rpc `

In this case, our RPC is 'get-bgp-summary-information'

Now, we add the first Junos-specific task to our playbook:

``` 
    - name: Get BGP summary info and saves it into a file
      juniper_junos_rpc:
        rpcs: "get-bgp-summary-information"
        formats: xml
        dest: "{{playbook_dir}}/tmp/{{ inventory_hostname }}_bgp.txt"
```

Now, let's go over the module arguments in this task:
* 'rpcs' the RPC we want to call
* 'formats' desired format of the reply
* 'dest' where to save the xml reply

Next, we need to extract what we want: BGP sessions state. To achieve this, we need to first study the XML reply structure. Again, we need some help from the CLI to understand what we'll be receiving.

` show bgp summary | display xml `

After running the command above, we get this (omitted) output:

``` 
<bgp-peer junos:style="terse">
            <peer-address>10.242.80.54</peer-address>
            <peer-as>64575</peer-as>
            <input-messages>312310</input-messages>
            <output-messages>1260015</output-messages>
            <route-queue-count>0</route-queue-count>
            <flap-count>2</flap-count>
            <elapsed-time junos:seconds="4882864">8w0d 12:21:04</elapsed-time>
            <peer-state junos:format="Establ">Established</peer-state>
            <bgp-rib junos:style="terse">
                <name>Homes-vrf.inet.0</name>
                <active-prefix-count>34</active-prefix-count>
                <received-prefix-count>36</received-prefix-count>
                <accepted-prefix-count>36</accepted-prefix-count>
                <suppressed-prefix-count>0</suppressed-prefix-count>
            </bgp-rib>
        </bgp-peer>
```

The actual output is pages upon pages long, but we have what we need here to identify the XML path for the peer-state node. In this case, it is:

` /bgp-information/bgp-peer/peer-state `

