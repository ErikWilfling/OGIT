# HR

## Overview

A human-resources department (HR department) of an organization performs human resource management, overseeing various aspects of employment, such as compliance with labor law and employment standards, administration of employee benefits, and some aspects of recruitment.

## Job evaluation

Entities and attributes directly under `ogit.HR:` cover the description and
evaluation of a position -- what a post is for, what it requires, what work it
comprises, and what pay grade that work justifies. The `Recruiting`
sub-namespace, by contrast, covers the candidate side (applicants, resumes,
skills, scores).

### Entities

| Entity | Purpose |
|---|---|
| `Position` | A post in the establishment plan, independent of its holder. |
| `JobDescription` | The documented account of a position; the evidentiary basis for evaluation. |
| `WorkUnit` | A body of work leading to one delimitable result, carrying a share of working time. |
| `QualificationRequirement` | A qualification the post requires -- degree, further training, knowledge, experience, licence. |
| `Occupation` | A recognised register entry: vocational training occupation, further-training occupation, or course of study. |
| `Authority` | A power delegated to the post holder -- signature, decision, directive, committee. |
| `Subordination` | A group of staff reporting directly to the post, with kind, headcount and grade. |
| `Grading` | The evaluation result: assigned grade, reasoning, evidence. |

### Attributes

| Attribute | On | Fixed values |
|---|---|---|
| `positionNumber` | `Position`, `JobDescription` | |
| `employmentFraction` | `Position` | |
| `timeShare` | `WorkUnit` | |
| `characteristicCode` | `WorkUnit` | |
| `qualificationKind` | `QualificationRequirement` | `degree`, `furtherTraining`, `professionalKnowledge`, `professionalExperience`, `licence` |
| `degreeLevel` | `QualificationRequirement` | `vocational`, `bachelor`, `master`, `diploma`, `stateExam`, `doctorate`, `other` |
| `institutionType` | `QualificationRequirement` | `university`, `universityOfAppliedSciences`, `vocationalSchool`, `other` |
| `fieldOfStudy`, `minimumExperience` | `QualificationRequirement` | |
| `occupationKind` | `Occupation` | `vocationalTraining`, `furtherTraining`, `courseOfStudy` |
| `occupationCode`, `trainingDuration` | `Occupation` | |
| `authorityKind` | `Authority` | `signature`, `decision`, `directive`, `committeeRepresentation` |
| `authorityScope` | `Authority` | `internal`, `external` |
| `signatureLevel` | `Authority` | `byOrder`, `perProcuration` |
| `authorityLimit` | `Authority` | |
| `subordinationKind` | `Subordination` | `functional`, `disciplinary`, `functionalAndDisciplinary` |
| `headcountFte` | `Subordination` | |
| `assignedGrade`, `rationale` | `Grading` | |
| `securingMode` | `Grading` | `derivable`, `statistical` |

### Shape

```
Organization <--belongs--  Position  <--describes--  JobDescription
                              |                            |
                         assignedTo                        +-- contains
                              v                            +-- requires --> QualificationRequirement
                        RoleAssignment                     +-- defines  --> Authority
                                                           +-- has      --> Subordination
                                                           +-- has      --> Grading
                                                           +-- represents --> Forms:Form
                                                           +-- relates  --> Automation:AutomationIssue

  QualificationRequirement -- relates --> Occupation        (names a register entry)
                           -- relates --> QualificationRequirement   (alternative / equivalent)

  WorkUnit, QualificationRequirement, Occupation, Authority, Subordination, Grading
        --classifiedUnder--> ClassificationStandardTreeBranch
                                        ^
                                        | classifiedUnder (already in OGIT)
                             Legal:LegalClause <--contains-- Legal:LegalNorm
                                        ^
                                        | references
                             Grading, Occupation
```

### Design notes

- **No new verbs.** Existing SGO verbs are reused throughout: `belongs`,
  `assignedTo`, `describes`, `contains`, `requires`, `defines`, `has`,
  `represents`, `relates`, `references`, plus
  `ClassificationStandard:classifiedUnder`. Verb restrictions in OGIT are
  source-side, so each entity that points at a tree branch declares the
  `classifiedUnder` pairing itself.
- **A qualification requirement can be stated two ways, and both are kept.**
  Naming a recognised occupation is a *reference* to a register entry that has
  its own curriculum, legal basis and duration -- collapsing it into a level
  plus a subject area loses exactly what evaluation needs. Broader wording
  ("a degree in economics or similar") has no register entry and is carried by
  the descriptive attributes instead. Hence `relates -> Occupation` alongside
  `degreeLevel` / `institutionType` / `fieldOfStudy`.
- **`relates` between two requirements means "either of these".** That is how an
  equivalent qualification held instead of the formally demanded one is
  expressed.
- **The classification standard is not duplicated here.** Pay-grade systems and
  their characteristics are instance data in a
  `ClassificationStandardTreeRoot` / `TreeBranch` tree; the norm text is
  `Legal:LegalNorm` / `LegalClause`. This namespace only points into them.
  `LegalClause` already declares `classifiedUnder` towards a tree branch, so
  the bridge between wording and category exists without change.
- **`Grading` is a node, not an attribute**, so an evaluation stays auditable
  and reproducible after the description changes, and a description can be
  graded more than once.
- **`timeShare` is what makes proportion rules computable** -- grading schemes
  commonly turn on the share of working time a characteristic accounts for.
  It sits on `WorkUnit` for that reason.
- Generic attributes (`ogit:name`, `description`, `content`, `status`,
  `validFrom`, `validTo`, `confidence`) are reused rather than redefined.
