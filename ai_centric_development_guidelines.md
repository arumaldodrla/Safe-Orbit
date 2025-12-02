# AI-Centric Development Guidelines: SafeOrbit

## 1. Introduction

These guidelines are intended for AI agents (such as Manus, Cursor, or Gemini) that will be involved in the development and maintenance of the SafeOrbit platform. By following these guidelines, AIs can effectively contribute to the project while ensuring code quality, consistency, and maintainability.

## 2. Reading the Documentation

Before writing any code, you must thoroughly read and understand the project documentation, including:

*   The Product Requirements Document (PRD)
*   The System Architecture Overview
*   The Detailed Component Design
*   The Data Model & Database Schema

This will ensure that you have a complete understanding of the project's goals, architecture, and technical specifications.

## 3. Generating Code and Infrastructure-as-Code

When generating code, you should:

*   Adhere to the coding conventions outlined below.
*   Write clean, modular, and well-documented code.
*   Generate infrastructure-as-code (e.g., Terraform for Supabase) to automate the provisioning and management of cloud resources.

## 4. Using MCP/Tooling

You will be provided with a set of tools (exposed via the Model Context Protocol, or MCP) that will allow you to interact with the SafeOrbit platform and its underlying infrastructure. You must use these tools to:

*   Read and write data to the Supabase database.
*   Call the APIs of external providers (Cloudflare, Bunny, etc.).
*   Execute scans and other automated tasks.

## 5. Proposing Changes and Requesting Human Approval

All code changes must be proposed via a pull request to the appropriate GitHub repository. For any action that is considered risky (e.g., a change that could affect production data or services), you must explicitly request human approval before proceeding.

## 6. Coding Conventions

*   **Language:** All code should be written in TypeScript.
*   **Project Structure:** The project will be structured as a monorepo, with separate packages for the frontend, backend, and shared libraries.
*   **Documentation:** All code should be documented using JSDoc comments. Additional documentation, such as architectural diagrams and design documents, should be written in Markdown.
