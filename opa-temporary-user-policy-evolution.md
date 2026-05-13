# OPA Policy Evolution for Temporary Users (Limited Tokens) - Monthly Analysis

## Executive Summary

This document traces the evolution of OPA (Open Policy Agent) policies for temporary users and limited tokens from 2023 to 2025, showing how their capabilities in the STEM app have changed over time. The analysis reveals a pattern of progressive feature enhancement followed by strategic restrictions and recent restoration of functionality.

## Current State (September 2025)

### Temporary User Capabilities
- **Library Access**: ✅ Full access (unlimited_library = true)
- **Track Generation**: ✅ Full access with `generation:limited` scope
- **Write Likes/Dislikes**: ✅ Can like/dislike objects (but UI shows signup prompt)
- **Read Likes/Dislikes**: ❌ Cannot retrieve their likes/dislikes
- **Trial Period**: 7 days from token creation
- **Token Scope**: `generation:limited likes:write`

## Monthly Evolution Timeline

### September 2025
**Commit**: `4758eaa` - "fix: add generation:limited scope support to Accounts API OPA policy"

**Key Changes**:
- Added support for `generation:limited` scope in `unlimited_generation` rule
- Restored full functionality for temporary users during trial period
- Fixed background task failures when saving stations to temporary user libraries

**Impact**: 
- ✅ **Restored**: Temporary users can now generate tracks and save stations
- ✅ **Fixed**: Recent tab now shows temporary user stations
- ✅ **Improved**: Background tasks no longer fail for temporary users

### September 2024
**Commit**: `fa4c391` - "feat: allow limited users to like/dislike objects, but ..."

**Key Changes**:
- Split `unlimited_likes` into separate `write_likes` and `read_likes` permissions
- Allowed temporary users to write likes/dislikes but not read them
- Added `likes:write` scope to temporary users

**Impact**:
- ✅ **Added**: Temporary users can like/dislike tracks and objects
- ❌ **Restricted**: Cannot retrieve their own likes/dislikes (encourages signup)
- 🔄 **UX**: Frontend shows signup prompt when liking (strategic friction)

### August 2024
**Commit**: `4477c08` - "feat: no library restrictions + set unlimited generation based on current time"

**Key Changes**:
- Removed all library access restrictions (`unlimited_library = true`)
- Added trial expiry date validation for generation permissions
- Simplified policy structure

**Impact**:
- ✅ **Removed**: All library access restrictions for any user type
- ✅ **Enhanced**: Trial expiry date now properly validates generation access
- 🔄 **Simplified**: Single policy for library access across all user types

**Commit**: `302fc26` - "feat: use generation:limited scope for unsubscribed users"

**Key Changes**:
- Changed temporary user tokens from `generation:unlimited` to `generation:limited`
- Updated Cognito token generation to add appropriate scopes based on subscription status

**Impact**:
- 🔄 **Scoped**: Temporary users now get `generation:limited` instead of unlimited
- 🔄 **Systematic**: Cognito now automatically assigns correct scopes based on user type

### July 2024
**Commit**: `e9c079f` - "feat: add read / write likes scope"

**Key Changes**:
- Changed scope from generic `likes` to specific `likes:write`
- Added `aws.cognito.signin.user.admin` fallback scope

**Impact**:
- 🔄 **Granular**: More specific scope naming for likes functionality
- 🔄 **Admin**: Added admin override capability

### March 2024
**Commit**: `8bcfa13` - "feat(auth): add scope for generation + likes"

**Key Changes**:
- Introduced scope-based permissions system
- Added `check_scope()` function with comma-separated scope parsing
- Added `generation:unlimited` and `likes` scopes
- Connected trial expiry date to generation permissions

**Impact**:
- ✅ **Foundation**: Established scope-based permission system
- ✅ **Granular**: Separate scopes for different capabilities
- ✅ **Trial**: Connected trial expiry to generation access

### June 2023
**Commits**: `0d67a52` and `86eaf85` - Library access changes for legacy users

**Key Changes**:
- Initially granted unlimited library access to legacy users
- Quickly reverted to limit legacy users' library access
- Established pattern of device owners and platform subscribers having unlimited access

**Impact**:
- 🔄 **Legacy**: Brief expansion then restriction of legacy user access
- 🔄 **Policy**: Established device/subscriber privilege pattern

## Scope Evolution

### Token Scopes by Period

| Period | Token Scopes | Library | Generation | Likes Write | Likes Read |
|--------|-------------|---------|------------|-------------|------------|
| **Sep 2025** | `generation:limited likes:write` | ✅ Full | ✅ Full | ✅ Yes | ❌ No |
| **Sep 2024** | `generation:limited likes:write` | ✅ Full | ❌ Limited | ✅ Yes | ❌ No |
| **Aug 2024** | `generation:limited likes:write` | ✅ Full | ❌ Limited | ✅ Yes | ❌ No |
| **Mar 2024** | `generation:unlimited likes` | ✅ Full | ✅ Full | ✅ Yes | ✅ Yes |
| **Pre-Mar 2024** | No scopes | ❌ Limited | ❌ No | ❌ No | ❌ No |

## Key Policy Rules Evolution

### `unlimited_generation` Rule
```rego
# September 2025 - Current
unlimited_generation {
    check_scope("generation:limited")  # ← Added for temporary users
}
unlimited_generation {
    check_scope("generation:unlimited") # ← For subscribers
}
unlimited_generation := true if {
    has_key(claims, "trial_expiry_date")
    claims.trial_expiry_date * ns > now  # ← Trial validation
}
```

### `write_likes` Rule (Split from unlimited_likes in Sep 2024)
```rego
write_likes {
    check_scope("likes:write")  # ← Temporary users have this
}
```

### `read_likes` Rule (New in Sep 2024)
```rego
read_likes {
    check_scope("likes:read")  # ← Temporary users DON'T have this
}
```

## Strategic Design Patterns

### 1. Progressive Enhancement (2023-2024)
- Started with no temporary user support
- Added scope-based permissions
- Granted full access to encourage engagement

### 2. Strategic Friction (2024)
- Split likes into read/write permissions
- Allow engagement but prevent retrieval
- UI prompts for signup when liking

### 3. Restoration & Stability (2025)
- Fixed broken functionality
- Maintained strategic restrictions
- Ensured consistent user experience

## Business Impact Analysis

### User Acquisition Funnel
1. **Entry**: Temporary users get 7-day trial with full library access
2. **Engagement**: Can generate tracks, save stations, like content
3. **Friction**: Cannot see their likes/dislikes (encourages signup)
4. **Conversion**: Must subscribe for unlimited generation after trial

### Technical Debt Patterns
- **September 2025 Fix**: Indicates that `generation:limited` scope wasn't properly supported
- **Policy Complexity**: Multiple overlapping rules for similar functionality
- **Test Coverage**: Limited session tests often commented out or marked as TODO

## Recommendations

### Immediate Actions
1. **Test Coverage**: Uncomment and update all limited session tests
2. **Documentation**: Update API documentation to reflect current scope behavior
3. **Monitoring**: Add metrics for temporary user conversion rates

### Future Considerations
1. **Scope Consolidation**: Consider simplifying overlapping permission rules
2. **Trial Analytics**: Track which features drive highest conversion
3. **UX Optimization**: A/B test different friction points for signup prompts

## Related Files
- `opa/policy/auth.rego` - Main policy definitions
- `opa/policy/auth_test.rego` - Test cases (many TODOs)
- `accounts_api/issue_token.py` - Token generation logic
- `cdk/lambda/cognito_pre_token_gen_trigger/` - Cognito scope assignment

## Conclusion

The evolution shows a maturing approach to temporary user onboarding - from no support to full access to strategic friction designed to optimize conversion while maintaining engagement. The September 2025 fixes indicate the system is now in a stable state where temporary users have meaningful trial access with clear upgrade incentives.

