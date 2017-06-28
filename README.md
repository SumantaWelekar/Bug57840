# Repro test case for Visual Studio for Mac bug [#57840][bug]

## Summary

This is a very severe and easily reproducible bug in Visual Studio for Mac
Version 7.1 Preview (7.1 build 1178). It resulted in VS for Mac reformatting
hundreds of source code files on disk without asking.

We have an existing PCL project, and I wanted to create a .NET Standard project
alongside it. When the .NET Standard project was added to the same directory
as the PCL project, all files (not just source code) were included in the .NET
Standard project. This is not itself much off a problem, except that the process
of adding these files to the project appears to ***open, reformat, and rewrite
to disk every single file***.

### There are two core issues here:

1. *none* of our source code should be opened and reformatted at all during
   the project creation process; this is just bad dangerous behavior, and also
   drastically slows down the project creation operation

2. The new project is following the default code formatting policy, and not
   the policy set on the solution, which in our case, is "Mono"

## Repro Steps

1. Create a new "Portable" library project and solution. Name the solution
  "PortableAndNetStandard" and the project "Portable".

2. Configure the solution's C# formatting policy to be "Mono" (e.g. 8-width
   tabs instead of 4-space "tabs").

3. Reformat "MyClass.cs" (_Edit → Format → Format Document_) so that it follows
   the "Mono" policy.

4. Add some more C# files to the project and notice they are in the "Mono"
   format.

5. Commit everything so far to a fresh git repository.

6. Create a new .NET Standard Library project (targeting 2.0 in my case) in the
   solution and name the project "NetStandard". The project creation wizard
   will note that the directory still exists, which is okay (and desired),
   so continue by answering "Yes".

7. Note that the "NetStandard" project now includes all files in the directory
   structure already (e.g. from the creation of our "Portable" project).

8. **The bugs:** note that all of the _source code has been reformatted_ using
   the default policy (not the "Mono" one configured for the solution).

## Analysis

Read each commit in this repository to see the effect of the bug on existing
projects. The addition of a new .NET Standard project to an existing solution
and directory structure already populated with code results in the rewriting of
every file.

Visual Studio should:

1. not open and rewrite _any_ existing files when creating a new project, ever
2. should always apply the solution level policy when formatting _at the
   appropriate time_ (e.g. when requested by the user).

[bug]: https://bugzilla.xamarin.com/show_bug.cgi?id=57840