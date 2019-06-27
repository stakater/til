# Jenkins Pipeline Removal Issue

## Overview

This document explains issue the issue that corrupted Jenkins setup and how the issue was resolved.

## Problem

Following issue were raised:

1-  Due to plugins update the pipeline used to get disappeared from time to time but once packages were updated pipelines restored.

2- In the last update we faced another issue that `Organizational folder` was showing empty although it should have shown the repositories that matches the regex in the github stakater organization.

3-  When jenkins version upgraded to > 2.159 it created a problem that it created a loop between keycloak and jenkins and will not show jenkins page. The issue caused the user to stuck in login loop.

## Solution

* `Issue 1 and 2`: was resolved by installing the plugin `workflow-cps:2.7.0` which was somehow missed during last update.

* `Issue-3`: was resolved by upgrading the `oic version from 1.4 to 1.6`.