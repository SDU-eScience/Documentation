UI
==

.. _WAYF:

WAYF
-----

We used WAYF service for connecting between our web based service and many provided institutions. The following image shows that how WAYF interacts between our web base service and the connected institutions.



.. figure::  images/WAFY.png

   :align:   center



1. The user accesses our web based service which requires a login. The user is directed to the WAYF website, where the user selects his/her institution.

2. If the user is not already logged in, the user is transferred to the institution’s login page which he/she has been selected.

3. After the user has logged into the institution, data about the user is sent to WAYF.

4. The WAYF website displays the user which personal data will be forwarded to our web based service. The user gives his/her consent by clicking on a button. And the user can tick a box to denote that this consent may be remembered for future visits to the same web based service.

5. WAYF sends the data to our web based service. If our web based service can approve the user on the basis of this data, access is granted.


.. _Ktor:

Ktor
------


.. _React:

React
------


