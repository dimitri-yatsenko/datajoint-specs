# **DataJoint Specs**

## **Purpose**  

The **DataJoint Specs** define the **standards, conventions, and best practices** for designing and managing **DataJoint pipelines**.
These specifications ensure that DataJoint implementations remain **consistent, scalable, and interoperable** across different scientific workflows and computing environments.

By following the DataJoint Specs, users and developers can: 
- Maintain **structured, reproducible data pipelines**.  
- Ensure **compatibility across different DataJoint implementations**.  
- Establish **best practices** for schema design, data integrity, and computational workflows.  
- Provide a **foundation for future enhancements** while preserving backward compatibility.  

## **Spec Versions & Implementation Versions**  
DataJoint Specs evolve **independently** from DataJoint implementations (e.g., [DataJoint Python](https://github.com/datajoint/datajont-python).
Each implementation adheres to a **specific spec version** to ensure compliance while allowing for gradual adoption of new features.

### **Versioning Strategy**  
| **Component** | **Purpose** | **Example Versions** |
|--------------|------------|----------------------|
| **DataJoint Specs** | Defines the standard for DataJoint pipeline structures and operations. | **v2.1, v2.2, v3.0** |
| **DataJoint Implementations** | Software libraries following the spec (e.g., DataJoint Python). | **0.16.0, 1.0.0, 2.0.0** |

### **Spec-to-Implementation Compatibility**  
Each implementation release specifies which **DataJoint Spec version** it supports.  
- **Patch Updates (e.g., v2.1 → v2.1.1)** – No implementation changes required unless fixing inconsistencies.  
- **Minor Updates (e.g., v2.1 → v2.2)** – Implementations gradually adopt new features while ensuring backward compatibility.  
- **Major Updates (e.g., v2.x → v3.0)** – May require **significant implementation changes** and deprecations.  

#### **Example Compatibility Table**  
| **Spec Version** | **DataJoint Python Version** | **Notes** |
|-----------------|-----------------------------|-----------|
| **v2.1** | **1.0.0** | First stable release under v2.x spec. |
| **v2.2** | **1.2.0** | Minor updates to storage handling. |
| **v3.0** | **2.0.0 (upcoming)** | Breaking changes requiring a major update. |

For implementation-specific details, refer to the **release notes of each DataJoint implementation**.

## **Enhancement Process (DSEPs)**  

Enhancements to the **DataJoint Specs** follow a structured **DataJoint Specs Enhancement Proposal (DSEP) process**.  
- All **proposed changes** (new features, clarifications, modifications) must be submitted as a **DSEP**.  
- Discussions, review, and approvals follow a **transparent, community-driven process**.  
- Accepted proposals are **documented and integrated** into future spec versions.  

For details, see the **[DSEP Process Document](DSEP_process.md)**.
