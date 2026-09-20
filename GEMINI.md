# DVLD Project Guidelines & Agent Instructions

## 1. Primary Knowledge Base (المرجع الأساسي للمشروع)
- Always read and consult [`PROJECT_REFERENCE.md`](file:///d:/Projects/DVLD/PROJECT_REFERENCE.md) before diving into deep codebase exploration.
- It contains the complete data dictionary, 3-tier architecture breakdown, business workflows, enums, UI controls mapping, and configuration settings.

## 2. Mandatory Maintenance Rule (قاعدة التحديث الإلزامي)
- Whenever you make ANY modification to the codebase (adding a form, editing business logic in `DVLD_Buisness`, modifying database queries or methods in `DVLD_DataAccess`, updating enums, or adding features):
  1. You MUST update [`PROJECT_REFERENCE.md`](file:///d:/Projects/DVLD/PROJECT_REFERENCE.md) to reflect the new state.
  2. Add an entry to the Changelog table in section 8 of `PROJECT_REFERENCE.md`.
  3. Never conclude a task without ensuring `PROJECT_REFERENCE.md` matches the actual codebase.

## 3. Build & Verification Standard
- Project solution: `DVLD/DVLD.sln`.
- Build command using Visual Studio MSBuild:
  ```powershell
  & "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" DVLD\DVLD.sln /p:Configuration=Debug
  ```
- Always verify that the solution builds with 0 errors after any code change.
