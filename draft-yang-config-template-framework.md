---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###

title: "Principle for populating list of YANG data nodes using a template technique."
abbrev: "yang-template-framework"
category: info

docname: draft-yang-config-template-framework
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: netmod
keyword:
 - templates
 - profiles
 - yang scalability
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 
 -
    fullname: Robert Peschi
    organization: Nokia
    city: Antwerp
    email: robert.peschi@nokia.com
 -
    fullname: Shiya Ashraf
    organization: Nokia
    city: Antwerp
    email: shiya.ashraf@nokia.com
 -
    fullname: Deepak Rajaram
    organization: Nokia
    city: Chennai
    email: deepak.rajaram@nokia.com
normative:

informative:


--- abstract

In YANG design, templates refer to pre-defined structures or configurations that can be applied multiple times, potentially with variations, to ensure efficiency in managing complex networks. These templates are essential when a network model requires repetitive configurations or shares a similar structural pattern across different network elements.

While YANG itself doesn’t have a formal "template" keyword or specific syntax dedicated to templates, developers often achieve template-like behaviour through YANG constructs such as grouping, uses, typedef, and augment. These constructs help modularize and standardize configurations.

In a typical deployment, A parent device can host multiple functional entities (e.g. a sub-systems) which is replicated many times, where each such functional instance is to be controlled by a set of data nodes. This can lead to long device configuration times, large memory footprint in the parent device and inefficient YANG validation procedures due to the large running data store size. This becomes more and more difficult as the system grows bigger and integrated, which leads individual data designers to employ various methods as per the requirements.

This document explores some of the constructs and highlights related issues while using such methods and possible solutions to mitigate such issues. 

This document presents a generic method that improves the efficiency issues mentioned above and which relies on features already available in the yang language that are supported by off-the-shelf toolset. It should be considered as a method that can be used when designing a module for a specific purpose, possibly a specific standard.  


--- middle

# Introduction

This draft considers the case of a device where each such functional instance is to be controlled by a set of data nodes with almost identical values from instance to instance. 

Rather than defining a YANG model where all data nodes of all instances need to be configured one by one in the device running data store by the management system (e.g. a NC client), this document presents the idea of a “generic data model” in which the running data store is configured with a list of instances without explicit data node being configured and the instruction to locally populate each instance's data nodes with a copy of a "model" data node set (called "template") that is configured in the running data store. 

Although the data nodes of instances have largely similar values across, a small number of data nodes may however need to have a value that differs from the ones in the common template. The generic method presented in this document also gives the possibility to overrule or complement values originating from the template, on a per instance basis.

# Concept

a)Identify the set of replicable data nodes                           
b)Configure a list of templates containing such data nodes            
c)Configure a list of instances, each of them referring to one of the configured templates along with any customization.                      
d)Parent device instantiating the required data, controls each instance, ie: merges the data from template and instance and produce the final set of configurations per instance, like Netconf merge.
     
     Eg:
     Identify common data nodes: 
     a, b, c

     Configure list of templates:
     Template common {
              container{
                        a=x
                        b=y
                        c=z
                       }
                     }
      Configure list of instances with template reference:
      Instance {
               Template-reference = “common”
               a=z
               b
               c
               }
      Parent device produces final configuration for each instance:
       Instance {
                 a=z
                 b=y
                 c=z
                }

# Template Framework

The templates concept can be achieved with existing yang constructs like grouping and uses statements.
 
 Example:
 
## Grouping construct
 
By defining common structures using grouping,you avoid repeating code,ensure consistency,and make future changes easier since modifications only need to happen in one place. 

 Example:
 
  grouping interface-config {
 
  leaf interface-name {
  
  type string;
  
  }
  
  leaf admin-status {
  
  type boolean;
  
  }
  
}

Here, the interface-config grouping defines a common structure that could be reused across different YANG modules or different parts of the same module.
 
## Uses construct
The "uses" statement applies a previously defined grouping. This allows the grouped set of definitions to be instantiated where needed in the model. 
 Example:
  container interfaces {
 
  list interface {
  
  key "interface-name";
  
  uses interface-config;
  
  }
  
 }
    
In this example, the uses statement applies the interface-config grouping defined earlier.
 
## Mandatories, Defaults, Leafrefs
While conceptually, the idea of templates improves the re-usability and consistency factor, there are certain nuances, which needs to be addressed while handling data nodes in the grouping with defaults and mandatories and leafrefs.

If the grouping created with replicable data nodes contains default or mandatory nodes, and such groupings are used both in the template and in the instance, there is a possibility of overwriting the configured value of the template with the default value in the instance due to the merge operation.

When using leafref inside a grouping (which acts as a template), Extra care must be taken on the path specified. This is because grouping is essentially a reusable blueprint, and the leafref path must be resolved correctly when the grouping is used in different contexts.

There are two primary approaches to using leafref in templates:
 
A)Absolute Paths: These reference a specific location in the YANG tree.
 
B)Relative Paths: These are defined relative to the location where the grouping is used.
 
While there is no recommended option to resolve the path, it is a designers choice to choose Absolute path where the referred path is a fixed well defined global path and relative path which could be flexible and re-usable depending on the context in which the groupings are defined.

## Refine construct
In order to solve the issue with defaults and mandatories, existing yang language provides us with a powerful construct: "refine"

The refine statement in YANG is used to modify or refine the definitions of a grouping when it is applied using the uses statement. This is particularly useful when the same template (or grouping) is reused in different contexts but needs to be adjusted slightly without altering the original grouping definition.
While the refine statement is powerful, it can only be applied within the context of a uses statement, and it allows fine-tuning of things like mandatory requirements, default values, descriptions, and other constraints.

The basic workflow for using refine with templates involves:
 
1.Defining a grouping(without mandatories and defaults), which can be (re)used in a template and in certain functional instance.

2.Apply the grouping with the uses statement.

3.Use the refine statement to modify specific characteristics of the applied grouping in a particular context(refine mandatories and defaults).

Example of Using refine with Templates

Step-1: Defining a Grouping. Here, a basic grouping for an interface configurationis defined devoid of mandatories and defaults
 
grouping interface-config {

    leaf interface-name {

       type string;

        description "Name of the interface.";

    }

    leaf admin-status {

        type boolean;

        description "Administrative status of the interface.";

    }

}

Step-2: Applying the Grouping with uses construct
 
container templates {

    list interface {

        key "interface-name";

        uses interface-config;

    }

}

At this point, the interface-config grouping can be (re)used for the templates list as well as functional instance list . refinement to this grouping can be done to suit specific use cases.

Step-3: Refining the Grouping:
 
 In certain contexts refinement would be required to the interface-config grouping:
 
1.Make the admin-status mandatory in templates.
2.Change the default value of admin-status in templates.

 
container templates {

    list interface {

        key "interface-name";

        uses interface-config {

            refine admin-status {

                mandatory true;

                default "true";

            }

        }

    }

}

In this example:

•The interface-config grouping is used within the templates container.

•The admin-status is made mandatory and its default value is set to true (instead of false).


# Benefits of Templates in YANG Design

1. Reusability:
   Templates encourage reusability by allowing common structures to be defined once and reused multiple times. This reduces duplication and simplifies model management.
2. Consistency:
   By using templates, similar configurations are defined and applied uniformly, leading to more consistent data modeling and network configuration management.
3. Simplified Maintenance:
   When a template is modified, all places where it is used will automatically reflect those changes, reducing the effort required to maintain and update the model.
4. Modularity:
   Templates promote a modular approach to YANG model design. By splitting the model into reusable parts, it becomes easier to scale and extend.

# Conclusion

Using templates (grouping and uses) in YANG allows for efficient reuse of model components. The refine statement provides flexibility to adjust certain properties like mandatory or default in specific contexts. leafref enables cross-referencing of leaf values within or across modules, with careful attention to path resolution. Proper use of these features helps create clean, modular, and maintainable YANG data models.



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
