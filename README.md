# SistemFis
Despacho
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistema de AlmacÃ©n, Pedidos Multi-Producto & CategorÃ­as - Laboratorios Fisa, C.A.</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- React 18 & React DOM -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  <!-- Babel Compiler -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen">
  <div id="root"></div>
  <script type="text/babel">
    const { useState, useEffect, useMemo } = React;

    // ESTRUCTURA FÃSICA DE 25 BLOQUES SEGÃšN RACK.PDF (1,594 PALETAS TOTALES)
    const RACK_STRUCTURE = [
      { id: 'B1', buildings: 14, levels: 5, total: 70, reserve: false },
      { id: 'B2', buildings: 16, levels: 5, total: 80, reserve: false },
      { id: 'B3', buildings: 16, levels: 5, total: 80, reserve: false },
      { id: 'B4', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B5', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B6', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B7', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B8', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B9', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B10', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B11', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B12', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B13', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B14', buildings: 18, levels: 1, total: 18, reserve: true }, // RESERVA NIVEL SUELO
      { id: 'B15', buildings: 18, levels: 5, total: 90, reserve: false },
      { id: 'B16', buildings: 6, levels: 5, total: 30, reserve: false },
      { id: 'B17', buildings: 6, levels: 1, total: 6, reserve: true },   // RESERVA NIVEL SUELO
      { id: 'B18', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B19', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B20', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B21', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B22', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B23', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B24', buildings: 8, levels: 5, total: 40, reserve: false },
      { id: 'B25', buildings: 8, levels: 5, total: 40, reserve: false },
    ];

    // CATÃLOGO BASE DE CATEGORÃAS INICIALES
    const INITIAL_CATEGORIES = [
      { id: 'CAT-1', name: 'Detergentes', description: 'Productos de limpieza y cuidado del hogar' },
      { id: 'CAT-2', name: 'Cuidado de pies', description: 'Talcos y desodorantes pÃ©dicos' },
      { id: 'CAT-3', name: 'CREMA MOLDEADORA', description: 'Fijadores y productos para peinar' },
      { id: 'CAT-4', name: 'Gel antibacterial', description: 'Desinfectantes e higiene personal' },
      { id: 'CAT-5', name: 'Colonia', description: 'Fragancias y colonias infantiles/adulto' },
      { id: 'CAT-6', name: 'Cremas/Spray para Peinar', description: 'Tratamientos capilares y cremas de peinar' },
      { id: 'CAT-7', name: 'RESERVA', description: 'Zona temporal de recepciÃ³n e inventario especial' }
    ];

    // CATÃLOGO BASE DE PRODUCTOS
    const INITIAL_PRODUCTS = [
      { code: '31032', name: 'WOOLITE DETERG. LIQ. PROT Y CUIDADO DP 1800ML', category: 'Detergentes' },
      { code: '31033', name: 'WOOLITE DETERG. LIQ. PROT Y CUIDADO 900ML', category: 'Detergentes' },
      { code: '16900', name: 'E.NIGHT TALCO DESOD. PARA PIES 30G', category: 'Cuidado de pies' },
      { code: '16901', name: 'E.NIGHT TALCO DESOD. PARA PIES 85G', category: 'Cuidado de pies' },
      { code: '16917', name: 'E NIGTH FOR MEN CREMA MOLDEADORA 250G', category: 'CREMA MOLDEADORA' },
      { code: '25051', name: 'E.NIGHT GEL ANTIBACTERIAL ALOE V 200G', category: 'Gel antibacterial' },
      { code: '25016', name: 'E.NIGHT NIÃ‘OS COLONIA SUAVE CARICIA 200CC', category: 'Colonia' },
      { code: '855', name: 'DRENE PROH CpP LISO 240CC', category: 'Cremas/Spray para Peinar' },
      { code: '2026', name: 'UNIDAD DE RECEPCION / RESERVA', category: 'RESERVA' }
    ];

    // CARGAS INICIALES EN MAPA DE RACKS
    const INITIAL_RACK_MAP = {
      'PT-1-1-1': { code: '31032', name: 'WOOLITE DETERG. LIQ. PROT Y CUIDADO DP 1800ML', qty: 120, lot: 'LOT-A1' },
      'PT-1-1-2': { code: '31032', name: 'WOOLITE DETERG. LIQ. PROT Y CUIDADO DP 1800ML', qty: 80, lot: 'LOT-A2' },
      'PT-1-2-1': { code: '16900', name: 'E.NIGHT TALCO DESOD. PARA PIES 30G', qty: 450, lot: 'LOT-B1' },
      'PT-14-1-1': { code: '2026', name: 'UNIDAD DE RECEPCION / RESERVA', qty: 1000, lot: 'RES-01' },
      'PT-17-1-1': { code: '25051', name: 'E.NIGHT GEL ANTIBACTERIAL ALOE V 200G', qty: 500, lot: 'LOT-C1' }
    };

    // USUARIOS BASE CON ROLES ACTUALIZADOS (MASTER, ADMIN, OPERATOR, AUDITOR)
    const INITIAL_USERS = [
      { code: 'MASTER-01', firstName: 'NÃ©stor', lastName: 'RamÃ­rez', idCard: 'V-00000000', name: 'NÃ©stor RamÃ­rez (MASTER)', role: 'MASTER', status: 'APPROVED', pass: 'master123' },
      { code: 'EMP-9921', firstName: 'Admin', lastName: 'General', idCard: 'V-11111111', name: 'Administrador Principal', role: 'ADMIN', status: 'APPROVED', pass: 'admin' },
      { code: 'EMP-8834', firstName: 'Juan', lastName: 'PÃ©rez', idCard: 'V-22222222', name: 'Juan PÃ©rez (Operador)', role: 'OPERATOR', status: 'APPROVED', pass: 'operador' },
      { code: 'AUD-5510', firstName: 'Carlos', lastName: 'Mendoza', idCard: 'V-33333333', name: 'Carlos Mendoza (Auditor Externo)', role: 'AUDITOR', status: 'APPROVED', pass: 'auditor' }
    ];

    function App() {
      const [currentUser, setCurrentUser] = useState(null);
      const [activeTab, setActiveTab] = useState('orders');
      const [categories, setCategories] = useState(INITIAL_CATEGORIES);
      const [products, setProducts] = useState(INITIAL_PRODUCTS);
      const [rackMap, setRackMap] = useState(INITIAL_RACK_MAP);
      const [users, setUsers] = useState(INITIAL_USERS);
      const [pendingUsers, setPendingUsers] = useState([]);
      const [orders, setOrders] = useState([]);
      const [movements, setMovements] = useState([]);
      const [transfers, setTransfers] = useState([]);
      const [auditLogs, setAuditLogs] = useState([]);
      const [inventoryAdjustLogs, setInventoryAdjustLogs] = useState([]);
      const [operatorStats, setOperatorStats] = useState({ 'EMP-8834': 0, 'EMP-9921': 0 });
      const [selectedBlock, setSelectedBlock] = useState('B1');

      // Estado Modal EdiciÃ³n de UbicaciÃ³n en Mapa
      const [editingSlot, setEditingSlot] = useState(null);
      const [slotForm, setSlotForm] = useState({ code: '', qty: 0, lot: '' });

      // Estado Modal EdiciÃ³n de Usuario (Exclusivo MASTER)
      const [editingUser, setEditingUser] = useState(null);
      const [userEditForm, setUserEditForm] = useState({ firstName: '', lastName: '', idCard: '', pass: '', role: 'OPERATOR' });

      // Estado Ajuste de Inventario (#Aumentar/Disminuir Inventario# - Exclusivo ADMIN/MASTER)
      const [adjProduct, setAdjProduct] = useState('');
      const [adjType, setAdjType] = useState('AUMENTAR'); // 'AUMENTAR' | 'DISMINUIR'
      const [adjQty, setAdjQty] = useState(1);
      const [adjTargetSlot, setAdjTargetSlot] = useState('PT-14-1-1'); // Para aumento en mapa
      const [adjComment, setAdjComment] = useState('');

      // Estado Login y Registro
      const [loginCode, setLoginCode] = useState('');
      const [loginPass, setLoginPass] = useState('');
      const [loginError, setLoginError] = useState('');
      const [isRegistering, setIsRegistering] = useState(false);
      const [regForm, setRegForm] = useState({ firstName: '', lastName: '', idCard: '', code: '', role: 'OPERATOR', pass: '' });

      // Estado CreaciÃ³n de Pedidos Multi-Producto
      const [orderNumberInput, setOrderNumberInput] = useState('22102026');
      const [orderItemsCart, setOrderItemsCart] = useState([]);
      const [selectedProdForCart, setSelectedProdForCart] = useState('');
      const [selectedQtyForCart, setSelectedQtyForCart] = useState(1);

      // Estado Despacho y AuditorÃ­a
      const [selectedOrderId, setSelectedOrderId] = useState('');
      const [dispatchSuccess, setDispatchSuccess] = useState(null);
      const [auditSelectedOrderId, setAuditSelectedOrderId] = useState('');
      const [auditComment, setAuditComment] = useState('');
      const [auditStatusDecision, setAuditStatusDecision] = useState('AUDITADO_APROBADO');

      // Estado Transferencias
      const [transferFrom, setTransferFrom] = useState('');
      const [transferTo, setTransferTo] = useState('');
      const [transferQty, setTransferQty] = useState(0);

      // Estado Datos Maestros
      const [masterSubTab, setMasterSubTab] = useState('products');
      const [newMasterProd, setNewMasterProd] = useState({ code: '', name: '', category: 'Detergentes' });
      const [newMasterCat, setNewMasterCat] = useState({ name: '', description: '' });

      useEffect(() => {
        if (window.lucide) window.lucide.createIcons();
      }, [currentUser, activeTab, selectedBlock, isRegistering, editingSlot, editingUser, orders, rackMap, masterSubTab]);

      // BASE DE DATOS MAESTRA CONSOLIDADA
      const masterDatabase = useMemo(() => {
        const stockMap = {};
        Object.values(rackMap).forEach(item => {
          if (item && item.code) {
            stockMap[item.code] = (stockMap[item.code] || 0) + Number(item.qty || 0);
          }
        });
        return products.map(p => ({
          ...p,
          totalQty: stockMap[p.code] || 0
        }));
      }, [products, rackMap]);

      // BASE DE DATOS DE CATEGORÃAS CON CONTEO DE PRODUCTOS
      const categoryDatabase = useMemo(() => {
        return categories.map(c => {
          const count = products.filter(p => p.category.toLowerCase() === c.name.toLowerCase()).length;
          return { ...c, productCount: count };
        });
      }, [categories, products]);

      // LOGIN DE USUARIO Y REDIRECCIÃ“N SEGÃšN ROL
      const handleLogin = (e) => {
        e.preventDefault();
        const user = users.find(u => u.code.toLowerCase() === loginCode.toLowerCase() && u.pass === loginPass);
        if (user) {
          if (user.status !== 'APPROVED') {
            setLoginError('Su usuario estÃ¡ pendiente de aprobaciÃ³n.');
            return;
          }
          setCurrentUser(user);
          // Redirecciones por rol
          if (user.role === 'AUDITOR') {
            setActiveTab('audit'); // AUDITOR solo puede ir a la pantalla de AuditorÃ­a
          } else if (user.role === 'MASTER') {
            setActiveTab('master_users');
          } else if (user.role === 'ADMIN') {
            setActiveTab('master');
          } else {
            setActiveTab('orders');
          }
          setLoginError('');
        } else {
          setLoginError('CÃ³digo o contraseÃ±a incorrectos.');
        }
      };

      const handleRegisterSubmit = (e) => {
        e.preventDefault();
        setPendingUsers([...pendingUsers, { ...regForm, status: 'PENDING', date: new Date().toLocaleString() }]);
        alert('Solicitud enviada a los administradores.');
        setIsRegistering(false);
      };

      // GESTIÃ“N MASTER DE USUARIOS (EdiciÃ³n de Nombre, CÃ©dula, ContraseÃ±a y Rol)
      const handleOpenEditUser = (u) => {
        setEditingUser(u);
        setUserEditForm({
          firstName: u.firstName || u.name.split(' ')[0] || '',
          lastName: u.lastName || u.name.split(' ')[1] || '',
          idCard: u.idCard || 'V-00000000',
          pass: u.pass || '',
          role: u.role || 'OPERATOR'
        });
      };

      const handleSaveUserMaster = (e) => {
        e.preventDefault();
        if (!editingUser) return;
        setUsers(users.map(u => {
          if (u.code === editingUser.code) {
            const fullName = `${userEditForm.firstName.trim()} ${userEditForm.lastName.trim()}`;
            return {
              ...u,
              firstName: userEditForm.firstName.trim(),
              lastName: userEditForm.lastName.trim(),
              name: fullName,
              idCard: userEditForm.idCard.trim(),
              pass: userEditForm.pass,
              role: userEditForm.role
            };
          }
          return u;
        }));
        setEditingUser(null);
        alert('InformaciÃ³n de usuario actualizada exitosamente por MASTER.');
      };

      // AJUSTE DIRECTO DE INVENTARIO (#Aumentar/Disminuir Inventario# - ADMIN/MASTER)
      const handleExecuteInventoryAdjustment = (e) => {
        e.preventDefault();
        if (!adjProduct || adjQty <= 0) {
          alert('Seleccione un producto y una cantidad vÃ¡lida mayor a 0.');
          return;
        }
        if (!adjComment.trim()) {
          alert('EL COMENTARIO ES OBLIGATORIO para justificar el ajuste de inventario.');
          return;
        }

        const prodObj = products.find(p => p.code === adjProduct);
        if (!prodObj) return;

        const updatedMap = { ...rackMap };

        if (adjType === 'DISMINUIR') {
          // Descontar la cantidad ingresada
          let remQty = Number(adjQty);
          Object.keys(updatedMap).forEach(loc => {
            if (remQty > 0 && updatedMap[loc] && updatedMap[loc].code === adjProduct) {
              if (updatedMap[loc].qty <= remQty) {
                remQty -= updatedMap[loc].qty;
                delete updatedMap[loc];
              } else {
                updatedMap[loc].qty -= remQty;
                remQty = 0;
              }
            }
          });
        } else {
          // AumentAR: AÃ±adir la cantidad ingresada a la ubicaciÃ³n seleccionada del mapa
          const target = adjTargetSlot.trim().toUpperCase() || 'PT-14-1-1';
          if (updatedMap[target]) {
            updatedMap[target].qty += Number(adjQty);
          } else {
            updatedMap[target] = {
              code: prodObj.code,
              name: prodObj.name,
              qty: Number(adjQty),
              lot: 'AJUSTE-ING'
            };
          }
        }

        setRackMap(updatedMap);
        setInventoryAdjustLogs([{
          id: Date.now(),
          type: adjType,
          productCode: prodObj.code,
          productName: prodObj.name,
          qty: Number(adjQty),
          user: currentUser.name,
          comment: adjComment.trim(),
          date: new Date().toLocaleString()
        }, ...inventoryAdjustLogs]);

        alert(`Ajuste de inventario (${adjType}) realizado exitosamente.`);
        setAdjQty(1);
        setAdjComment('');
      };

      // AGREGAR ARTÃCULO AL CARRITO DEL PEDIDO MULTI-PRODUCTO
      const handleAddCartItem = (e) => {
        e.preventDefault();
        if (!selectedProdForCart || selectedQtyForCart <= 0) return;
        const prodObj = products.find(p => p.code === selectedProdForCart);
        if (!prodObj) return;

        const exists = orderItemsCart.find(i => i.code === selectedProdForCart);
        if (exists) {
          setOrderItemsCart(orderItemsCart.map(i => i.code === selectedProdForCart ? { ...i, qty: i.qty + Number(selectedQtyForCart) } : i));
        } else {
          setOrderItemsCart([...orderItemsCart, { code: prodObj.code, name: prodObj.name, qty: Number(selectedQtyForCart) }]);
        }
        setSelectedProdForCart('');
        setSelectedQtyForCart(1);
      };

      const handleRemoveCartItem = (code) => {
        setOrderItemsCart(orderItemsCart.filter(i => i.code !== code));
      };

      // GUARDAR Y REGISTRAR PEDIDO COMPLETO
      const handleFinalizeOrder = (e) => {
        e.preventDefault();
        const orderNum = orderNumberInput.trim();
        if (!orderNum) {
          alert('Debe especificar un nÃºmero de pedido (Ej: 22102026).');
          return;
        }
        if (orderItemsCart.length === 0) {
          alert('Debe incluir al menos un cÃ³digo de producto en el pedido.');
          return;
        }

        const orderExists = orders.some(o => o.orderNumber === orderNum && o.status !== 'COMPLETADO');
        if (orderExists) {
          alert(`El nÃºmero de pedido [${orderNum}] ya existe y se encuentra activo.`);
          return;
        }

        const newOrder = {
          id: `PED-${orderNum}`,
          orderNumber: orderNum,
          items: orderItemsCart,
          status: 'PENDIENTE',
          createdBy: currentUser.name,
          date: new Date().toLocaleTimeString(),
          auditComments: [],
          auditStatus: 'SIN_AUDITAR'
        };

        setOrders([newOrder, ...orders]);
        setOrderItemsCart([]);
        setOrderNumberInput('');
        alert(`Pedido #${orderNum} creado exitosamente. Estado: PENDIENTE (Esperando aprobaciÃ³n del Admin).`);
      };

      // APROBACIÃ“N DE PEDIDO POR ADMIN / MASTER
      const handleApproveOrder = (orderId) => {
        setOrders(orders.map(o => o.id === orderId ? { ...o, status: 'APROBADO' } : o));
      };

      // ELIMINACIÃ“N DE PEDIDO POR ADMIN / MASTER
      const handleDeleteOrder = (orderId) => {
        if (confirm('Â¿EstÃ¡ seguro de que desea eliminar este pedido?')) {
          setOrders(orders.filter(o => o.id !== orderId));
        }
      };

      // ENVIAR PEDIDO A LA COLA DE AUDITORÃA
      const handleSendToAudit = (orderId) => {
        setOrders(orders.map(o => o.id === orderId ? { ...o, status: 'EN_AUDITORIA' } : o));
        alert('Pedido enviado formalmente a la cola de AuditorÃ­a Externa.');
      };

      // PROCESAR INSPECCIÃ“N Y COMENTARIOS DE AUDITORÃA (AUDITOR / MASTER)
      const handleProcessAudit = (e) => {
        e.preventDefault();
        if (!auditSelectedOrderId) {
          alert('Seleccione un pedido para auditar.');
          return;
        }
        if (!auditComment.trim()) {
          alert('Debe ingresar un comentario u observaciÃ³n de la inspecciÃ³n fÃ­sica.');
          return;
        }

        const order = orders.find(o => o.id === auditSelectedOrderId);
        if (!order) return;

        const newAuditRecord = {
          auditor: currentUser.name,
          decision: auditStatusDecision,
          comment: auditComment.trim(),
          date: new Date().toLocaleString()
        };

        setOrders(orders.map(o => {
          if (o.id === auditSelectedOrderId) {
            return {
              ...o,
              status: auditStatusDecision === 'AUDITADO_APROBADO' ? 'AUDITADO_APROBADO' : 'CON_INCIDENCIA',
              auditStatus: auditStatusDecision,
              auditComments: [...(o.auditComments || []), newAuditRecord]
            };
          }
          return o;
        }));

        setAuditLogs([ { id: Date.now(), orderNum: order.orderNumber, ...newAuditRecord }, ...auditLogs ]);
        alert(`AuditorÃ­a registrada para el Pedido #${order.orderNumber}. Resultado: ${auditStatusDecision}`);
        setAuditSelectedOrderId('');
        setAuditComment('');
      };

      // DESPACHO FINAL
      const handleExecuteDispatch = () => {
        const order = orders.find(o => o.id === selectedOrderId);
        if (!order || (order.status !== 'APROBADO' && order.status !== 'AUDITADO_APROBADO')) {
          alert('Seleccione un pedido en estado APROBADO o AUDITADO_APROBADO.');
          return;
        }

        const copyMap = { ...rackMap };
        order.items.forEach(item => {
          let remQty = item.qty;
          Object.keys(copyMap).forEach(loc => {
            if (remQty > 0 && copyMap[loc] && copyMap[loc].code === item.code) {
              if (copyMap[loc].qty <= remQty) {
                remQty -= copyMap[loc].qty;
                delete copyMap[loc];
              } else {
                copyMap[loc].qty -= remQty;
                remQty = 0;
              }
            }
          });
        });

        setRackMap(copyMap);
        setOrders(orders.map(o => o.id === order.id ? { ...o, status: 'COMPLETADO' } : o));

        setOperatorStats(prev => ({
          ...prev,
          [currentUser.code]: (prev[currentUser.code] || 0) + 1
        }));

        setMovements([{
          id: Date.now(),
          orderId: order.orderNumber,
          itemsCount: order.items.length,
          operator: currentUser.name,
          date: new Date().toLocaleTimeString()
        }, ...movements]);

        setDispatchSuccess(`Despacho ejecutado exitosamente para el Pedido #${order.orderNumber}.`);
        setSelectedOrderId('');
      };

      // REGISTRAR NUEVO PRODUCTO EN DATOS MAESTROS (REGLA ANTI-DUPLICADOS)
      const handleCreateMasterProduct = (e) => {
        e.preventDefault();
        const codeTrim = newMasterProd.code.trim();
        const nameTrim = newMasterProd.name.trim();

        if (products.some(p => p.code.toLowerCase() === codeTrim.toLowerCase())) {
          alert(`ERROR: El cÃ³digo [${codeTrim}] ya existe en los Datos Maestros.`);
          return;
        }
        if (products.some(p => p.name.toLowerCase() === nameTrim.toLowerCase())) {
          alert(`ERROR: El nombre de producto [${nameTrim}] ya existe en los Datos Maestros.`);
          return;
        }

        setProducts([...products, { code: codeTrim, name: nameTrim, category: newMasterProd.category }]);
        setNewMasterProd({ code: '', name: '', category: categories[0]?.name || 'Detergentes' });
        alert('Producto registrado en los Datos Maestros.');
      };

      // REGISTRAR NUEVA CATEGORÃA EN DATOS MAESTROS (REGLA ANTI-DUPLICADOS)
      const handleCreateMasterCategory = (e) => {
        e.preventDefault();
        const catNameTrim = newMasterCat.name.trim();
        if (categories.some(c => c.name.toLowerCase() === catNameTrim.toLowerCase())) {
          alert(`ERROR: La categorÃ­a [${catNameTrim}] ya existe.`);
          return;
        }

        const newCatObj = {
          id: `CAT-${Date.now()}`,
          name: catNameTrim,
          description: newMasterCat.description.trim() || 'Sin descripciÃ³n'
        };

        setCategories([...categories, newCatObj]);
        setNewMasterCat({ name: '', description: '' });
        alert(`CategorÃ­a "${catNameTrim}" registrada con Ã©xito.`);
      };

      // EDICIÃ“N DIRECTA DE CASILLA EN MAPA (Solo ADMIN/MASTER/OPERATOR segÃºn permisos)
      const handleSaveSlotEdit = (e) => {
        e.preventDefault();
        if (!editingSlot) return;

        if (currentUser.role === 'OPERATOR') {
          alert('El usuario OPERATOR solo tiene permisos de lectura e ingreso en mapas, no de alteraciÃ³n de casillas.');
          setEditingSlot(null);
          return;
        }

        if (!slotForm.code || slotForm.qty <= 0) {
          const updated = { ...rackMap };
          delete updated[editingSlot];
          setRackMap(updated);
        } else {
          const prodObj = products.find(p => p.code === slotForm.code);
          setRackMap({
            ...rackMap,
            [editingSlot]: {
              code: slotForm.code,
              name: prodObj ? prodObj.name : 'Producto',
              qty: Number(slotForm.qty),
              lot: slotForm.lot || 'LOTE-MANUAL'
            }
          });
        }
        setEditingSlot(null);
      };

      // TRANSFERENCIAS
      const handleExecuteTransfer = (e) => {
        e.preventDefault();
        const fromLoc = transferFrom.trim().toUpperCase();
        const toLoc = transferTo.trim().toUpperCase();

        if (!rackMap[fromLoc]) {
          alert(`UbicaciÃ³n origen [${fromLoc}] sin carga.`);
          return;
        }

        const item = rackMap[fromLoc];
        const qtyToMove = transferQty > 0 && transferQty <= item.qty ? transferQty : item.qty;

        const updated = { ...rackMap };
        if (item.qty - qtyToMove <= 0) delete updated[fromLoc];
        else updated[fromLoc].qty -= qtyToMove;

        updated[toLoc] = updated[toLoc] 
          ? { ...updated[toLoc], qty: updated[toLoc].qty + qtyToMove }
          : { ...item, qty: qtyToMove };

        setRackMap(updated);
        setTransfers([{ id: Date.now(), from: fromLoc, to: toLoc, product: item.name, qty: qtyToMove, user: currentUser.name, date: new Date().toLocaleTimeString() }, ...transfers]);
        alert(`Transferencia realizada de ${fromLoc} a ${toLoc}.`);
        setTransferFrom(''); setTransferTo(''); setTransferQty(0);
      };

      // MODAL / PANTALLA LOGIN
      if (!currentUser) {
        return (
          <div className="flex items-center justify-center min-h-screen p-4 bg-slate-950">
            <div className="w-full max-w-md bg-slate-900 border border-slate-800 rounded-2xl p-6 shadow-2xl">
              <div className="text-center mb-6">
                <div className="inline-flex p-3 bg-blue-600/20 text-blue-400 rounded-xl mb-3">
                  <i data-lucide="package-search" className="w-8 h-8"></i>
                </div>
                <h1 className="text-2xl font-bold text-white">Laboratorios Fisa, C.A.</h1>
                <p className="text-slate-400 text-sm mt-1">GestiÃ³n de AlmacÃ©n WMS & Control de Pedidos</p>
              </div>

              {!isRegistering ? (
                <form onSubmit={handleLogin} className="space-y-4">
                  <div>
                    <label className="block text-xs font-semibold text-slate-400 uppercase mb-1">CÃ³digo de Usuario</label>
                    <input type="text" placeholder="Ej: MASTER-01, EMP-9921, AUD-5510" value={loginCode} onChange={e => setLoginCode(e.target.value)} className="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-2.5 text-white" required />
                  </div>
                  <div>
                    <label className="block text-xs font-semibold text-slate-400 uppercase mb-1">ContraseÃ±a</label>
                    <input type="password" placeholder="â€¢â€¢â€¢â€¢â€¢â€¢â€¢â€¢" value={loginPass} onChange={e => setLoginPass(e.target.value)} className="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-2.5 text-white" required />
                  </div>
                  {loginError && <p className="text-rose-400 text-xs font-medium">{loginError}</p>}
                  <button type="submit" className="w-full bg-blue-600 hover:bg-blue-500 text-white font-semibold py-2.5 rounded-xl transition-all">Iniciar SesiÃ³n</button>
                  <div className="text-center pt-3 border-t border-slate-800">
                    <button type="button" onClick={() => setIsRegistering(true)} className="text-sm text-blue-400 hover:underline">Â¿Nuevo trabajador? Solicitar Registro</button>
                  </div>
                </form>
              ) : (
                <form onSubmit={handleRegisterSubmit} className="space-y-3">
                  <h2 className="text-lg font-bold text-white mb-2">Solicitud de Registro</h2>
                  <div className="grid grid-cols-2 gap-2">
                    <input type="text" placeholder="Nombre" onChange={e => setRegForm({...regForm, firstName: e.target.value})} className="bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white" required />
                    <input type="text" placeholder="Apellido" onChange={e => setRegForm({...regForm, lastName: e.target.value})} className="bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white" required />
                  </div>
                  <input type="text" placeholder="CÃ©dula" onChange={e => setRegForm({...regForm, idCard: e.target.value})} className="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white" required />
                  <input type="text" placeholder="CÃ³digo Empleado" onChange={e => setRegForm({...regForm, code: e.target.value})} className="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white" required />
                  <select onChange={e => setRegForm({...regForm, role: e.target.value})} className="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white">
                    <option value="OPERATOR">Operador</option>
                    <option value="AUDITOR">Auditor (Interno / Externo)</option>
                    <option value="ADMIN">Administrador</option>
                  </select>
                  <input type="password" placeholder="ContraseÃ±a" onChange={e => setRegForm({...regForm, pass: e.target.value})} className="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm text-white" required />
                  <div className="flex gap-2 pt-2">
                    <button type="button" onClick={() => setIsRegistering(false)} className="w-1/2 bg-slate-800 text-slate-300 py-2 rounded-lg text-sm">Cancelar</button>
                    <button type="submit" className="w-1/2 bg-emerald-600 text-white py-2 rounded-lg text-sm font-semibold">Enviar</button>
                  </div>
                </form>
              )}
            </div>
          </div>
        );
      }

      return (
        <div className="flex flex-col min-h-screen">
          {/* HEADER */}
          <header className="bg-slate-900 border-b border-slate-800 px-4 py-3 flex items-center justify-between">
            <div className="flex items-center gap-3">
              <div className="p-2 bg-blue-600 rounded-lg text-white">
                <i data-lucide="boxes" className="w-5 h-5"></i>
              </div>
              <div>
                <h1 className="font-bold text-white leading-none">Laboratorios Fisa, C.A.</h1>
                <span className="text-xs text-slate-400">
                  Usuario: <strong className="text-blue-400">{currentUser.name}</strong> | Nivel: <span className="bg-slate-800 text-amber-400 px-2 py-0.5 rounded text-[10px] font-bold border border-amber-500/30">{currentUser.role}</span>
                </span>
              </div>
            </div>
            <button onClick={() => setCurrentUser(null)} className="p-2 bg-slate-800 hover:bg-slate-700 text-rose-400 rounded-lg text-sm flex items-center gap-1">
              <i data-lucide="log-out" className="w-4 h-4"></i> Salir
            </button>
          </header>

          {/* MENÃš NAVEGACIÃ“N - RESTRINGIDO SEGÃšN ROL */}
          <nav className="bg-slate-900/60 border-b border-slate-800 px-4 flex gap-2 overflow-x-auto">
            {/* AUDITOR SOLO TIENE ACCESO A SU PANTALLA DE AUDITORÃA */}
            {currentUser.role === 'AUDITOR' ? (
              <button onClick={() => setActiveTab('audit')} className="py-3 px-4 text-sm font-bold border-b-2 border-purple-500 text-purple-400 flex items-center gap-2">
                <i data-lucide="shield-check" className="w-4 h-4"></i> MÃ³dulo de AuditorÃ­a Externa / InspecciÃ³n
              </button>
            ) : (
              <>
                {/* PANTALLA EXCLUSIVA MASTER */}
                {currentUser.role === 'MASTER' && (
                  <button onClick={() => setActiveTab('master_users')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'master_users' ? 'border-rose-500 text-rose-400 font-bold' : 'border-transparent text-slate-400'}`}>
                    <i data-lucide="crown" className="w-4 h-4"></i> GestiÃ³n Master de Usuarios
                  </button>
                )}

                <button onClick={() => setActiveTab('orders')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'orders' ? 'border-amber-500 text-amber-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="clipboard-list" className="w-4 h-4"></i> Pedidos Multi-Producto
                </button>

                <button onClick={() => setActiveTab('audit')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'audit' ? 'border-purple-500 text-purple-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="shield-check" className="w-4 h-4"></i> AuditorÃ­a Externa & InspecciÃ³n
                </button>

                <button onClick={() => setActiveTab('output')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'output' ? 'border-blue-500 text-blue-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="qr-code" className="w-4 h-4"></i> Despacho / Salidas
                </button>

                {(currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
                  <button onClick={() => setActiveTab('inventory_adjust')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'inventory_adjust' ? 'border-emerald-500 text-emerald-400 font-bold' : 'border-transparent text-slate-400'}`}>
                    <i data-lucide="sliders" className="w-4 h-4"></i> #Aumentar/Disminuir Inventario#
                  </button>
                )}

                <button onClick={() => setActiveTab('master')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'master' ? 'border-emerald-500 text-emerald-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="database" className="w-4 h-4"></i> Datos Maestros & CategorÃ­as
                </button>

                <button onClick={() => setActiveTab('map')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'map' ? 'border-blue-500 text-blue-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="map" className="w-4 h-4"></i> Mapa de Racks
                </button>

                <button onClick={() => setActiveTab('transfer')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'transfer' ? 'border-purple-500 text-purple-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="arrow-left-right" className="w-4 h-4"></i> Transferencias
                </button>

                <button onClick={() => setActiveTab('performance')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'performance' ? 'border-cyan-500 text-cyan-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="trending-up" className="w-4 h-4"></i> DesempeÃ±o
                </button>

                <button onClick={() => setActiveTab('kpi')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'kpi' ? 'border-blue-500 text-blue-400 font-bold' : 'border-transparent text-slate-400'}`}>
                  <i data-lucide="bar-chart-3" className="w-4 h-4"></i> Indicadores & Paletas
                </button>

                {(currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
                  <button onClick={() => setActiveTab('approvals')} className={`py-3 px-4 text-sm font-medium border-b-2 whitespace-nowrap flex items-center gap-2 ${activeTab === 'approvals' ? 'border-amber-500 text-amber-400 font-bold' : 'border-transparent text-slate-400'}`}>
                    <i data-lucide="user-check" className="w-4 h-4"></i> Aprobar Usuarios ({pendingUsers.length})
                  </button>
                )}
              </>
            )}
          </nav>

          {/* CONTENIDO PRINCIPAL */}
          <main className="flex-1 p-4 md:p-6 max-w-7xl mx-auto w-full">
            
            {/* PANTALLA EXCLUSIVA MASTER: GESTIÃ“N DE USUARIOS */}
            {activeTab === 'master_users' && currentUser.role === 'MASTER' && (
              <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-6">
                <div className="flex justify-between items-center">
                  <div>
                    <h2 className="text-xl font-bold text-rose-400 flex items-center gap-2">
                      <i data-lucide="crown"></i> Panel de Control Master - Usuarios y PermisologÃ­a
                    </h2>
                    <p className="text-xs text-slate-400">VisualizaciÃ³n universal y ediciÃ³n de credenciales, documentos y niveles de acceso.</p>
                  </div>
                </div>

                <div className="overflow-x-auto">
                  <table className="w-full text-left text-xs text-slate-300">
                    <thead className="bg-slate-950 text-slate-400 border-b border-slate-800">
                      <tr>
                        <th className="p-3">CÃ³digo</th>
                        <th className="p-3">Nombre Completo</th>
                        <th className="p-3">CÃ©dula / Documento</th>
                        <th className="p-3">Rol / Nivel</th>
                        <th className="p-3">Estado</th>
                        <th className="p-3 text-right">Acciones MASTER</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-800">
                      {users.map(u => (
                        <tr key={u.code} className="hover:bg-slate-800/50">
                          <td className="p-3 font-bold text-blue-400">{u.code}</td>
                          <td className="p-3 font-semibold text-white">{u.name}</td>
                          <td className="p-3 text-slate-300">{u.idCard || 'N/A'}</td>
                          <td className="p-3">
                            <span className={`px-2 py-1 rounded font-bold text-[10px] ${
                              u.role === 'MASTER' ? 'bg-rose-950 text-rose-400 border border-rose-600/40' :
                              u.role === 'ADMIN' ? 'bg-amber-950 text-amber-400 border border-amber-600/40' :
                              u.role === 'AUDITOR' ? 'bg-purple-950 text-purple-400 border border-purple-600/40' :
                              'bg-slate-800 text-slate-300'
                            }`}>
                              {u.role}
                            </span>
                          </td>
                          <td className="p-3"><span className="text-emerald-400 font-bold">{u.status}</span></td>
                          <td className="p-3 text-right">
                            <button onClick={() => handleOpenEditUser(u)} className="bg-rose-600 hover:bg-rose-500 text-white px-3 py-1.5 rounded-lg font-bold text-xs">
                              Editar Permisos & Datos
                            </button>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>

                {/* MODAL EDITAR USUARIO MASTER */}
                {editingUser && (
                  <div className="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4 z-50">
                    <form onSubmit={handleSaveUserMaster} className="bg-slate-900 border border-slate-800 rounded-2xl p-6 max-w-md w-full shadow-2xl space-y-4">
                      <h3 className="text-lg font-bold text-rose-400 flex items-center gap-2">
                        <i data-lucide="user-cog"></i> Editar Usuario: {editingUser.code}
                      </h3>
                      <div className="grid grid-cols-2 gap-2">
                        <div>
                          <label className="block text-xs font-semibold text-slate-400 mb-1">Nombre</label>
                          <input type="text" value={userEditForm.firstName} onChange={e => setUserEditForm({...userEditForm, firstName: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                        </div>
                        <div>
                          <label className="block text-xs font-semibold text-slate-400 mb-1">Apellido</label>
                          <input type="text" value={userEditForm.lastName} onChange={e => setUserEditForm({...userEditForm, lastName: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                        </div>
                      </div>
                      <div>
                        <label className="block text-xs font-semibold text-slate-400 mb-1">CÃ©dula / Documento Identidad</label>
                        <input type="text" value={userEditForm.idCard} onChange={e => setUserEditForm({...userEditForm, idCard: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                      </div>
                      <div>
                        <label className="block text-xs font-semibold text-slate-400 mb-1">ContraseÃ±a</label>
                        <input type="text" value={userEditForm.pass} onChange={e => setUserEditForm({...userEditForm, pass: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                      </div>
                      <div>
                        <label className="block text-xs font-semibold text-slate-400 mb-1">Nivel / Rol de Usuario</label>
                        <select value={userEditForm.role} onChange={e => setUserEditForm({...userEditForm, role: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white font-bold">
                          <option value="OPERATOR">OPERATOR (Usuario Operario)</option>
                          <option value="AUDITOR">AUDITOR (Auditor Externo/Interno)</option>
                          <option value="ADMIN">ADMIN (Administrador)</option>
                          <option value="MASTER">MASTER (Controlador Absoluto)</option>
                        </select>
                      </div>
                      <div className="flex gap-2 pt-2">
                        <button type="button" onClick={() => setEditingUser(null)} className="w-1/2 bg-slate-800 text-slate-300 py-2.5 rounded-xl text-xs font-bold">Cancelar</button>
                        <button type="submit" className="w-1/2 bg-rose-600 hover:bg-rose-500 text-white py-2.5 rounded-xl text-xs font-bold">Guardar Cambios MASTER</button>
                      </div>
                    </form>
                  </div>
                )}
              </div>
            )}

            {/* PANTALLA ÃšNICA Y EXCLUSIVA PARA AUDITORES / AUDITORÃA EXTERNA */}
            {activeTab === 'audit' && (
              <div className="space-y-6">
                <div className="bg-slate-900 border border-purple-900/50 rounded-2xl p-5 shadow-xl space-y-4">
                  <h2 className="text-xl font-bold text-purple-400 flex items-center gap-2">
                    <i data-lucide="shield-check"></i> Pantalla de AuditorÃ­a Externa & InspecciÃ³n FÃ­sica
                  </h2>
                  <p className="text-xs text-slate-400">MÃ³dulo exclusivo de verificaciÃ³n de cantidades fÃ­sicas, registro de incidencias y comentarios oficiales de auditores.</p>

                  <form onSubmit={handleProcessAudit} className="grid grid-cols-1 md:grid-cols-2 gap-4 bg-slate-950 p-4 rounded-xl border border-slate-800">
                    <div className="space-y-3">
                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">Seleccionar Pedido en cola de AuditorÃ­a</label>
                        <select value={auditSelectedOrderId} onChange={e => setAuditSelectedOrderId(e.target.value)} className="w-full bg-slate-900 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required>
                          <option value="">Seleccione Pedido para Auditar...</option>
                          {orders.filter(o => o.status === 'EN_AUDITORIA' || o.status === 'APROBADO').map(o => (
                            <option key={o.id} value={o.id}>Pedido #{o.orderNumber} ({o.items.length} Ã­tems) - Creado por: {o.createdBy}</option>
                          ))}
                        </select>
                      </div>

                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">Resultado de InspecciÃ³n FÃ­sica</label>
                        <select value={auditStatusDecision} onChange={e => setAuditStatusDecision(e.target.value)} className="w-full bg-slate-900 border border-slate-800 rounded-xl p-2.5 text-xs font-bold text-white">
                          <option value="AUDITADO_APROBADO">AUDITADO_APROBADO (Visto Bueno Conforme)</option>
                          <option value="CON_INCIDENCIA">CON_INCIDENCIA (Diferencia de Cantidad / Falta)</option>
                        </select>
                      </div>
                    </div>

                    <div className="space-y-3">
                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">Comentario u Observaciones del Auditor (OBLIGATORIO)</label>
                        <textarea rows="3" value={auditComment} onChange={e => setAuditComment(e.target.value)} placeholder="Ej: Cantidades verificadas fÃ­sicamente en rampa 2. Se observa 1 caja averiada..." className="w-full bg-slate-900 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                      </div>
                      <button type="submit" className="w-full bg-purple-600 hover:bg-purple-500 text-white font-bold py-2.5 rounded-xl transition-all">
                        Emitir Dictamen de AuditorÃ­a
                      </button>
                    </div>
                  </form>
                </div>

                {/* DETALLE Y ESTADO DE AUDITORÃAS REGISTRADAS */}
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h3 className="text-md font-bold text-white mb-3">Historial de Revisiones e Incidencias</h3>
                  <div className="space-y-3">
                    {auditLogs.length === 0 ? (
                      <p className="text-slate-500 text-xs italic">No se han registrado auditorÃ­as todavÃ­a.</p>
                    ) : (
                      auditLogs.map(a => (
                        <div key={a.id} className="p-3 bg-slate-950 border border-slate-800 rounded-xl text-xs space-y-1">
                          <div className="flex justify-between items-center">
                            <span className="font-bold text-purple-400">Pedido #{a.orderNum}</span>
                            <span className={`px-2 py-0.5 rounded font-bold text-[10px] ${a.decision === 'AUDITADO_APROBADO' ? 'bg-emerald-950 text-emerald-400' : 'bg-rose-950 text-rose-400'}`}>
                              {a.decision}
                            </span>
                          </div>
                          <p className="text-slate-300 italic">"{a.comment}"</p>
                          <div className="flex justify-between text-slate-500 text-[10px] pt-1">
                            <span>Auditor: {a.auditor}</span>
                            <span>{a.date}</span>
                          </div>
                        </div>
                      ))
                    )}
                  </div>
                </div>
              </div>
            )}

            {/* PANTALLA EXCLUSIVA ADMIN/MASTER: #AUMENTAR / DISMINUIR INVENTARIO# */}
            {activeTab === 'inventory_adjust' && (currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
              <div className="grid md:grid-cols-2 gap-6">
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-4">
                  <h2 className="text-lg font-bold text-emerald-400 flex items-center gap-2">
                    <i data-lucide="sliders"></i> Ajuste Directo de Inventario (#Aumentar/Disminuir#)
                  </h2>
                  <p className="text-xs text-slate-400">Permite ajustar cantidades en inventario y actualizar el mapa de racks automÃ¡ticamente con comentario obligatorio.</p>

                  <form onSubmit={handleExecuteInventoryAdjustment} className="space-y-3">
                    <div>
                      <label className="block text-xs font-bold text-slate-300 mb-1">Producto a Ajustar</label>
                      <select value={adjProduct} onChange={e => setAdjProduct(e.target.value)} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required>
                        <option value="">Seleccione Producto...</option>
                        {products.map(p => (
                          <option key={p.code} value={p.code}>[{p.code}] {p.name}</option>
                        ))}
                      </select>
                    </div>

                    <div className="grid grid-cols-2 gap-2">
                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">Tipo de Ajuste</label>
                        <select value={adjType} onChange={e => setAdjType(e.target.value)} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white font-bold">
                          <option value="AUMENTAR">+ AUMENTAR INVENTARIO</option>
                          <option value="DISMINUIR">- DISMINUIR INVENTARIO</option>
                        </select>
                      </div>
                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">Cantidad de Unidades</label>
                        <input type="number" min="1" value={adjQty} onChange={e => setAdjQty(Number(e.target.value))} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white font-bold" required />
                      </div>
                    </div>

                    {adjType === 'AUMENTAR' && (
                      <div>
                        <label className="block text-xs font-bold text-slate-300 mb-1">UbicaciÃ³n Destino en Mapa (Ej: PT-14-1-1)</label>
                        <input type="text" value={adjTargetSlot} onChange={e => setAdjTargetSlot(e.target.value)} placeholder="Ej: PT-14-1-1" className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white font-bold" required />
                      </div>
                    )}

                    <div>
                      <label className="block text-xs font-bold text-slate-300 mb-1">Comentario / Motivo del Ajuste (OBLIGATORIO)</label>
                      <textarea rows="3" value={adjComment} onChange={e => setAdjComment(e.target.value)} placeholder="Ej: Ajuste por merma, rotura o recepciÃ³n extra no programada..." className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-xs text-white" required />
                    </div>

                    <button type="submit" className="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-xl transition-all">
                      Ejecutar Ajuste de Inventario
                    </button>
                  </form>
                </div>

                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-4">
                  <h2 className="text-lg font-bold text-white flex items-center gap-2">
                    <i data-lucide="history"></i> BitÃ¡cora de Ajustes de Inventario
                  </h2>
                  <div className="space-y-3 max-h-[420px] overflow-y-auto pr-1">
                    {inventoryAdjustLogs.length === 0 ? (
                      <p className="text-slate-500 text-xs italic">No hay registros de ajuste manual.</p>
                    ) : (
                      inventoryAdjustLogs.map(log => (
                        <div key={log.id} className="p-3 bg-slate-950 border border-slate-800 rounded-xl text-xs space-y-1">
                          <div className="flex justify-between items-center">
                            <span className="font-bold text-white">[{log.productCode}] {log.productName}</span>
                            <span className={`px-2 py-0.5 rounded font-bold text-[10px] ${log.type === 'AUMENTAR' ? 'bg-emerald-950 text-emerald-400' : 'bg-rose-950 text-rose-400'}`}>
                              {log.type === 'AUMENTAR' ? `+${log.qty} und` : `-${log.qty} und`}
                            </span>
                          </div>
                          <p className="text-slate-300 italic">"{log.comment}"</p>
                          <div className="flex justify-between text-slate-500 text-[10px] pt-1">
                            <span>Por: {log.user}</span>
                            <span>{log.date}</span>
                          </div>
                        </div>
                      ))
                    )}
                  </div>
                </div>
              </div>
            )}

            {/* VISTA 1: CREACIÃ“N DE PEDIDOS MULTI-PRODUCTO */}
            {activeTab === 'orders' && (
              <div className="grid md:grid-cols-2 gap-6">
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-4">
                  <h2 className="text-lg font-bold text-white flex items-center gap-2">
                    <i data-lucide="plus-circle" className="text-amber-400"></i> Registrar Pedido Multi-Producto
                  </h2>
                  <div>
                    <label className="block text-xs font-medium text-slate-400 mb-1">NÃºmero de Pedido (Ej: 22102026)</label>
                    <input type="text" value={orderNumberInput} onChange={e => setOrderNumberInput(e.target.value)} placeholder="Ej: 22102026" className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-white text-sm font-bold" required />
                  </div>

                  <div className="p-3 bg-slate-950 border border-slate-800 rounded-xl space-y-3">
                    <span className="text-xs font-semibold text-amber-400 uppercase block">Agregar CÃ³digos y Cantidades al Pedido</span>
                    
                    <div className="grid grid-cols-1 sm:grid-cols-3 gap-2">
                      <select value={selectedProdForCart} onChange={e => setSelectedProdForCart(e.target.value)} className="sm:col-span-2 bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs text-white">
                        <option value="">Seleccione Producto...</option>
                        {products.map(p => (
                          <option key={p.code} value={p.code}>[{p.code}] {p.name}</option>
                        ))}
                      </select>
                      <input type="number" min="1" value={selectedQtyForCart} onChange={e => setSelectedQtyForCart(Number(e.target.value))} className="bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs text-white" placeholder="Cant" />
                    </div>
                    <button onClick={handleAddCartItem} type="button" className="w-full bg-slate-800 hover:bg-slate-700 text-amber-300 py-2 rounded-lg text-xs font-bold transition-all">
                      + AÃ±adir Producto al Pedido
                    </button>
                  </div>

                  <div className="space-y-2">
                    <span className="text-xs font-semibold text-slate-400 block">Detalle de ArtÃ­culos Incluidos ({orderItemsCart.length}):</span>
                    {orderItemsCart.length === 0 ? (
                      <p className="text-slate-500 text-xs italic">AÃºn no ha aÃ±adido productos a este pedido.</p>
                    ) : (
                      <div className="space-y-1.5 max-h-48 overflow-y-auto pr-1">
                        {orderItemsCart.map(item => (
                          <div key={item.code} className="p-2 bg-slate-950 border border-slate-800 rounded-lg flex justify-between items-center text-xs">
                            <div>
                              <span className="font-bold text-white block">[{item.code}] {item.name}</span>
                              <span className="text-amber-400 font-semibold">{item.qty} unidades</span>
                            </div>
                            <button onClick={() => handleRemoveCartItem(item.code)} className="text-rose-400 hover:underline text-xs">Quitar</button>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                  <button onClick={handleFinalizeOrder} className="w-full bg-amber-600 hover:bg-amber-500 text-white font-bold py-3 rounded-xl transition-all">
                    Guardar Pedido Multi-Producto
                  </button>
                </div>

                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                    <i data-lucide="clipboard-check" className="text-amber-400"></i> Pedidos Registrados & Flujo de Estado
                  </h2>
                  <div className="space-y-3 max-h-[480px] overflow-y-auto pr-1">
                    {orders.length === 0 ? (
                      <p className="text-slate-500 text-sm italic">No hay pedidos registrados en el sistema.</p>
                    ) : (
                      orders.map(o => (
                        <div key={o.id} className="p-3 bg-slate-950 border border-slate-800 rounded-xl space-y-2 text-xs">
                          <div className="flex justify-between items-center border-b border-slate-800 pb-2">
                            <div>
                              <span className="font-bold text-amber-400 text-sm block">Pedido #{o.orderNumber}</span>
                              <span className="text-slate-400">Creado por: {o.createdBy} ({o.date})</span>
                            </div>
                            <div className="flex items-center gap-1">
                              {/* OPCIONES ADMIN / MASTER */}
                              {(currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
                                <>
                                  {o.status === 'PENDIENTE' && (
                                    <button onClick={() => handleApproveOrder(o.id)} className="bg-emerald-600 hover:bg-emerald-500 text-white font-bold px-2.5 py-1 rounded text-xs">Aprobar</button>
                                  )}
                                  <button onClick={() => handleDeleteOrder(o.id)} className="bg-rose-800 hover:bg-rose-700 text-white px-2 py-1 rounded text-xs">Eliminar</button>
                                </>
                              )}

                              {/* BANDERAS DE ESTADO */}
                              {o.status === 'PENDIENTE' && <span className="bg-amber-950 text-amber-400 px-2 py-1 rounded border border-amber-600/40">PENDIENTE</span>}
                              {o.status === 'APROBADO' && (
                                <div className="flex items-center gap-1">
                                  <span className="bg-emerald-950 text-emerald-400 px-2 py-1 rounded border border-emerald-600/40 font-bold">APROBADO</span>
                                  <button onClick={() => handleSendToAudit(o.id)} className="bg-purple-700 hover:bg-purple-600 text-white px-2 py-1 rounded font-bold text-[10px]">Enviar a AuditorÃ­a</button>
                                </div>
                              )}
                              {o.status === 'EN_AUDITORIA' && <span className="bg-purple-950 text-purple-300 px-2 py-1 rounded border border-purple-500/40 font-bold animate-pulse">EN AUDITORÃA</span>}
                              {o.status === 'AUDITADO_APROBADO' && <span className="bg-emerald-900 text-emerald-200 px-2 py-1 rounded font-bold">AUDITADO CONFORME</span>}
                              {o.status === 'CON_INCIDENCIA' && <span className="bg-rose-950 text-rose-300 px-2 py-1 rounded font-bold border border-rose-600/40">CON INCIDENCIA</span>}
                              {o.status === 'COMPLETADO' && <span className="bg-slate-800 text-slate-400 px-2 py-1 rounded">DESPACHADO</span>}
                            </div>
                          </div>
                          <div className="space-y-1">
                            <span className="text-[10px] text-slate-500 block uppercase font-bold">ArtÃ­culos del Pedido:</span>
                            {o.items.map((it, idx) => (
                              <div key={idx} className="flex justify-between text-slate-300 bg-slate-900/60 p-1.5 rounded">
                                <span>[{it.code}] {it.name}</span>
                                <span className="font-bold text-emerald-400">{it.qty} und</span>
                              </div>
                            ))}
                          </div>
                        </div>
                      ))
                    )}
                  </div>
                </div>
              </div>
            )}

            {/* VISTA 2: DESPACHO / SALIDAS */}
            {activeTab === 'output' && (
              <div className="grid md:grid-cols-2 gap-6">
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                    <i data-lucide="qr-code" className="text-blue-400"></i> Ejecutar Despacho mediante Pedido Autorizado
                  </h2>
                  <div className="space-y-4">
                    <div>
                      <label className="block text-xs font-medium text-slate-400 mb-1">Seleccionar Pedido Listo para Despacho</label>
                      <select value={selectedOrderId} onChange={e => setSelectedOrderId(e.target.value)} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-white text-sm">
                        <option value="">Seleccione Pedido Aprobado / Auditado...</option>
                        {orders.filter(o => o.status === 'APROBADO' || o.status === 'AUDITADO_APROBADO').map(o => (
                          <option key={o.id} value={o.id}>Pedido #{o.orderNumber} ({o.items.length} productos) - [{o.status}]</option>
                        ))}
                      </select>
                    </div>
                    <button onClick={handleExecuteDispatch} className="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-xl flex items-center justify-center gap-2">
                      <i data-lucide="barcode"></i> Procesar Despacho & Descontar Inventario
                    </button>
                    {dispatchSuccess && <div className="p-3 bg-emerald-950 border border-emerald-500/50 text-emerald-300 rounded-xl text-xs font-semibold">{dispatchSuccess}</div>}
                  </div>
                </div>
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                    <i data-lucide="history" className="text-blue-400"></i> Historial de Despachos Ejecutados
                  </h2>
                  <div className="space-y-3 max-h-[380px] overflow-y-auto pr-1">
                    {movements.map(m => (
                      <div key={m.id} className="p-3 bg-slate-950 border border-slate-800 rounded-xl text-xs space-y-1">
                        <div className="flex justify-between font-bold text-slate-200">
                          <span>Pedido #{m.orderId}</span>
                          <span className="text-emerald-400">{m.itemsCount} producto(s) procesado(s)</span>
                        </div>
                        <div className="flex justify-between text-slate-400">
                          <span>Operador: {m.operator}</span>
                          <span>{m.date}</span>
                        </div>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            )}

            {/* VISTA 3: DATOS MAESTROS & CATEGORÃAS */}
            {activeTab === 'master' && (
              <div className="space-y-6">
                <div className="flex gap-2 border-b border-slate-800 pb-2">
                  <button onClick={() => setMasterSubTab('products')} className={`px-4 py-2 rounded-xl text-xs font-bold transition-all ${masterSubTab === 'products' ? 'bg-emerald-600 text-white' : 'bg-slate-900 text-slate-400 hover:text-white'}`}>
                    Base de Datos de Productos
                  </button>
                  <button onClick={() => setMasterSubTab('categories')} className={`px-4 py-2 rounded-xl text-xs font-bold transition-all ${masterSubTab === 'categories' ? 'bg-emerald-600 text-white' : 'bg-slate-900 text-slate-400 hover:text-white'}`}>
                    Base de Datos de CategorÃ­as ({categories.length})
                  </button>
                </div>

                {masterSubTab === 'products' && (
                  <div className="space-y-6">
                    {/* REGISTRO SOLO PARA ADMIN O MASTER */}
                    {(currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
                      <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                        <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                          <i data-lucide="plus-circle" className="text-emerald-400"></i> Registrar Producto (Regla Anti-Duplicados Activa)
                        </h2>
                        <form onSubmit={handleCreateMasterProduct} className="grid grid-cols-1 md:grid-cols-4 gap-3">
                          <input type="text" placeholder="CÃ³digo Ãšnico" value={newMasterProd.code} onChange={e => setNewMasterProd({...newMasterProd, code: e.target.value})} className="bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white" required />
                          <input type="text" placeholder="Nombre Ãšnico de Producto" value={newMasterProd.name} onChange={e => setNewMasterProd({...newMasterProd, name: e.target.value})} className="bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white" required />
                          <select value={newMasterProd.category} onChange={e => setNewMasterProd({...newMasterProd, category: e.target.value})} className="bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white">
                            {categories.map(c => (
                              <option key={c.id} value={c.name}>{c.name}</option>
                            ))}
                          </select>
                          <button type="submit" className="bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-2.5 rounded-xl transition-all">Registrar Producto</button>
                        </form>
                      </div>
                    )}

                    <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                      <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                        <i data-lucide="database" className="text-emerald-400"></i> Base de Datos Unificada de Productos & Suma de Ubicaciones
                      </h2>
                      <div className="overflow-x-auto">
                        <table className="w-full text-left text-xs text-slate-300">
                          <thead className="bg-slate-950 text-slate-400 border-b border-slate-800">
                            <tr>
                              <th className="p-3">CÃ³digo</th>
                              <th className="p-3">Nombre del Producto</th>
                              <th className="p-3">CategorÃ­a</th>
                              <th className="p-3 text-right">Cantidad Total (Suma de Ubicaciones)</th>
                            </tr>
                          </thead>
                          <tbody className="divide-y divide-slate-800">
                            {masterDatabase.map(p => (
                              <tr key={p.code} className="hover:bg-slate-800/50">
                                <td className="p-3 font-bold text-blue-400">{p.code}</td>
                                <td className="p-3 font-semibold text-white">{p.name}</td>
                                <td className="p-3"><span className="bg-slate-800 px-2 py-1 rounded text-slate-300">{p.category}</span></td>
                                <td className="p-3 text-right font-black text-emerald-400 text-sm">{p.totalQty.toLocaleString()} und</td>
                              </tr>
                            ))}
                          </tbody>
                        </table>
                      </div>
                    </div>
                  </div>
                )}

                {masterSubTab === 'categories' && (
                  <div className="space-y-6">
                    {(currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
                      <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                        <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                          <i data-lucide="folder-plus" className="text-emerald-400"></i> Crear Nueva CategorÃ­a
                        </h2>
                        <form onSubmit={handleCreateMasterCategory} className="grid grid-cols-1 md:grid-cols-3 gap-3">
                          <input type="text" placeholder="Nombre de la CategorÃ­a" value={newMasterCat.name} onChange={e => setNewMasterCat({...newMasterCat, name: e.target.value})} className="bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white" required />
                          <input type="text" placeholder="DescripciÃ³n breve" value={newMasterCat.description} onChange={e => setNewMasterCat({...newMasterCat, description: e.target.value})} className="bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white" />
                          <button type="submit" className="bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-2.5 rounded-xl transition-all">Registrar CategorÃ­a</button>
                        </form>
                      </div>
                    )}

                    <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                      <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                        <i data-lucide="layers" className="text-emerald-400"></i> Base de Datos de CategorÃ­as Registradas
                      </h2>
                      <div className="overflow-x-auto">
                        <table className="w-full text-left text-xs text-slate-300">
                          <thead className="bg-slate-950 text-slate-400 border-b border-slate-800">
                            <tr>
                              <th className="p-3">ID CategorÃ­a</th>
                              <th className="p-3">Nombre CategorÃ­a</th>
                              <th className="p-3">DescripciÃ³n</th>
                              <th className="p-3 text-right">Productos Asociados</th>
                            </tr>
                          </thead>
                          <tbody className="divide-y divide-slate-800">
                            {categoryDatabase.map(c => (
                              <tr key={c.id} className="hover:bg-slate-800/50">
                                <td className="p-3 font-bold text-slate-500">{c.id}</td>
                                <td className="p-3 font-semibold text-white">{c.name}</td>
                                <td className="p-3 text-slate-400">{c.description}</td>
                                <td className="p-3 text-right font-black text-amber-400">{c.productCount} producto(s)</td>
                              </tr>
                            ))}
                          </tbody>
                        </table>
                      </div>
                    </div>
                  </div>
                )}
              </div>
            )}

            {/* VISTA 4: MAPA DE RACKS */}
            {activeTab === 'map' && (
              <div className="space-y-4">
                <div className="bg-slate-900 border border-slate-800 p-4 rounded-2xl flex items-center justify-between flex-wrap gap-4">
                  <div>
                    <h2 className="text-lg font-bold text-white">Mapa FÃ­sico de Racks (25 Bloques / 1,594 Paletas)</h2>
                    <p className="text-xs text-slate-400">Haz clic en una casilla para ver/modificar contenido (Permisos segÃºn Nivel).</p>
                  </div>
                  <select value={selectedBlock} onChange={e => setSelectedBlock(e.target.value)} className="bg-slate-950 border border-slate-800 text-white rounded-xl p-2.5 text-sm">
                    {RACK_STRUCTURE.map(b => (
                      <option key={b.id} value={b.id}>{b.id} - {b.reserve ? 'RESERVA' : `${b.buildings} Edificios`} [{b.total} Paletas]</option>
                    ))}
                  </select>
                </div>

                {(() => {
                  const blk = RACK_STRUCTURE.find(b => b.id === selectedBlock);
                  const blockNum = blk.id.replace('B', '');
                  return (
                    <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                      <div className="grid grid-cols-2 sm:grid-cols-4 md:grid-cols-6 lg:grid-cols-9 gap-2">
                        {Array.from({ length: blk.buildings }).map((_, eIdx) => (
                          <div key={eIdx} className="bg-slate-950 border border-slate-800 p-2 rounded-xl text-center">
                            <span className="text-[10px] font-bold text-slate-400 block mb-1">E{eIdx + 1}</span>
                            <div className="space-y-1">
                              {Array.from({ length: blk.levels }).map((_, lIdx) => {
                                const levelNum = blk.levels - lIdx;
                                const slotCode = `PT-${blockNum}-${eIdx + 1}-${levelNum}`;
                                const slotData = rackMap[slotCode];
                                return (
                                  <button 
                                    key={lIdx}
                                    onClick={() => {
                                      setEditingSlot(slotCode);
                                      setSlotForm({ code: slotData ? slotData.code : '', qty: slotData ? slotData.qty : 0, lot: slotData ? slotData.lot : '' });
                                    }}
                                    className={`w-full text-[9px] py-1.5 rounded font-semibold transition-all block border ${
                                      slotData 
                                        ? 'bg-purple-950 border-purple-500 text-purple-200 hover:bg-purple-900' 
                                        : 'bg-slate-900 border-slate-800 text-slate-400 hover:bg-slate-800'
                                    }`}
                                  >
                                    N{levelNum} {slotData ? `[${slotData.qty}u]` : 'LIBRE'}
                                  </button>
                                );
                              })}
                            </div>
                          </div>
                        ))}
                      </div>
                    </div>
                  );
                })()}

                {/* MODAL CASILLA DEL MAPA */}
                {editingSlot && (
                  <div className="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4 z-50">
                    <form onSubmit={handleSaveSlotEdit} className="bg-slate-900 border border-slate-800 rounded-2xl p-6 max-w-md w-full shadow-2xl space-y-4">
                      <h3 className="text-lg font-bold text-white">Detalle / AsignaciÃ³n en Paleta: <span className="text-purple-400">{editingSlot}</span></h3>
                      
                      {currentUser.role === 'OPERATOR' && (
                        <p className="text-xs text-amber-400 bg-amber-950/50 p-2 rounded border border-amber-500/30">
                          Modo Lectura Activo (El rol OPERATOR no puede alterar casillas directamente desde el mapa).
                        </p>
                      )}

                      <div>
                        <label className="block text-xs font-medium text-slate-400 mb-1">CÃ³digo de Producto</label>
                        <select disabled={currentUser.role === 'OPERATOR'} value={slotForm.code} onChange={e => setSlotForm({...slotForm, code: e.target.value})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white disabled:opacity-50">
                          <option value="">Vaciar Paleta / Sin Carga</option>
                          {products.map(p => (
                            <option key={p.code} value={p.code}>[{p.code}] {p.name}</option>
                          ))}
                        </select>
                      </div>
                      <div>
                        <label className="block text-xs font-medium text-slate-400 mb-1">Cantidad de Unidades</label>
                        <input disabled={currentUser.role === 'OPERATOR'} type="number" min="0" value={slotForm.qty} onChange={e => setSlotForm({...slotForm, qty: Number(e.target.value)})} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white disabled:opacity-50" />
                      </div>
                      <div>
                        <label className="block text-xs font-medium text-slate-400 mb-1">NÃºmero de Lote</label>
                        <input disabled={currentUser.role === 'OPERATOR'} type="text" value={slotForm.lot} onChange={e => setSlotForm({...slotForm, lot: e.target.value})} placeholder="Ej: LOTE-2026-X" className="w-full bg-slate-950 border border-slate-800 rounded-xl p-2.5 text-sm text-white disabled:opacity-50" />
                      </div>
                      <div className="flex gap-2 pt-2">
                        <button type="button" onClick={() => setEditingSlot(null)} className="w-1/2 bg-slate-800 text-slate-300 py-2.5 rounded-xl text-sm font-semibold">Cerrar</button>
                        {currentUser.role !== 'OPERATOR' && (
                          <button type="submit" className="w-1/2 bg-purple-600 hover:bg-purple-500 text-white py-2.5 rounded-xl text-sm font-bold">Guardar Cambios</button>
                        )}
                      </div>
                    </form>
                  </div>
                )}
              </div>
            )}

            {/* VISTA 5: TRANSFERENCIAS */}
            {activeTab === 'transfer' && (
              <div className="grid md:grid-cols-2 gap-6">
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                    <i data-lucide="arrow-left-right" className="text-purple-400"></i> Transferir Carga de UbicaciÃ³n
                  </h2>
                  <form onSubmit={handleExecuteTransfer} className="space-y-4">
                    <input type="text" placeholder="UbicaciÃ³n Origen (Ej: PT-1-1-1)" value={transferFrom} onChange={e => setTransferFrom(e.target.value)} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-white text-sm" required />
                    <input type="text" placeholder="UbicaciÃ³n Destino (Ej: PT-3-2-5)" value={transferTo} onChange={e => setTransferTo(e.target.value)} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-white text-sm" required />
                    <input type="number" min="0" placeholder="Cantidad a mover (0 = Toda)" value={transferQty} onChange={e => setTransferQty(Number(e.target.value))} className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-white text-sm" />
                    <button type="submit" className="w-full bg-purple-600 hover:bg-purple-500 text-white font-bold py-3 rounded-xl transition-all">
                      Ejecutar ReubicaciÃ³n & Actualizar Mapa
                    </button>
                  </form>
                </div>
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl">
                  <h2 className="text-lg font-bold text-white mb-4 flex items-center gap-2">
                    <i data-lucide="list" className="text-purple-400"></i> Registro de Reubicaciones
                  </h2>
                  <div className="space-y-3 max-h-[380px] overflow-y-auto pr-1">
                    {transfers.map(t => (
                      <div key={t.id} className="p-3 bg-slate-950 border border-slate-800 rounded-xl text-xs space-y-1">
                        <div className="flex justify-between font-bold text-purple-300">
                          <span>{t.product}</span>
                          <span>{t.qty} und</span>
                        </div>
                        <div className="flex justify-between text-slate-400">
                          <span>De: <strong className="text-rose-400">{t.from}</strong> âž” A: <strong className="text-emerald-400">{t.to}</strong></span>
                          <span>Por: {t.user}</span>
                        </div>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            )}

            {/* VISTA 6: DESEMPEÃ‘O POR OPERADOR */}
            {activeTab === 'performance' && (
              <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-6">
                <h2 className="text-lg font-bold text-white flex items-center gap-2">
                  <i data-lucide="trending-up" className="text-cyan-400"></i> GrÃ¡fica de DesempeÃ±o por Operador
                </h2>
                <div className="space-y-4">
                  {users.map(u => {
                    const count = operatorStats[u.code] || 0;
                    const maxVal = Math.max(...Object.values(operatorStats), 1);
                    const pct = Math.round((count / maxVal) * 100);
                    return (
                      <div key={u.code} className="space-y-1">
                        <div className="flex justify-between text-xs font-semibold text-slate-300">
                          <span>{u.name} ({u.code}) - {u.role}</span>
                          <span className="text-cyan-400 font-bold">{count} Despachos Realizados</span>
                        </div>
                        <div className="w-full bg-slate-950 rounded-full h-4 border border-slate-800 overflow-hidden">
                          <div className="bg-cyan-500 h-full rounded-full transition-all duration-500" style={{ width: `${Math.max(pct, 5)}%` }}></div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              </div>
            )}

            {/* VISTA 7: KPI & PALETAS */}
            {activeTab === 'kpi' && (
              <div className="space-y-6">
                <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                  <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl shadow-xl">
                    <span className="text-xs font-semibold text-slate-400 block uppercase">Capacidad Total (Paletas)</span>
                    <span className="text-3xl font-black text-blue-400 mt-2 block">1,594 Ubicaciones</span>
                  </div>
                  <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl shadow-xl">
                    <span className="text-xs font-semibold text-slate-400 block uppercase">Paletas Ocupadas</span>
                    <span className="text-3xl font-black text-purple-400 mt-2 block">{Object.keys(rackMap).length} Paletas</span>
                  </div>
                  <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl shadow-xl">
                    <span className="text-xs font-semibold text-slate-400 block uppercase">Paletas Libres</span>
                    <span className="text-3xl font-black text-emerald-400 mt-2 block">{1594 - Object.keys(rackMap).length} Paletas</span>
                  </div>
                </div>

                <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl shadow-xl space-y-4">
                  <h3 className="text-md font-bold text-white">Inventario Detallado por Producto</h3>
                  <div className="space-y-3">
                    {masterDatabase.map(p => {
                      const maxUnits = Math.max(...masterDatabase.map(m => m.totalQty), 1);
                      const barWidth = Math.round((p.totalQty / maxUnits) * 100);
                      return (
                        <div key={p.code} className="space-y-1">
                          <div className="flex justify-between text-xs text-slate-300">
                            <span>[{p.code}] {p.name}</span>
                            <span className="font-bold text-emerald-400">{p.totalQty} und</span>
                          </div>
                          <div className="w-full bg-slate-950 rounded-full h-3 border border-slate-800 overflow-hidden">
                            <div className="bg-emerald-500 h-full rounded-full transition-all duration-500" style={{ width: `${Math.max(barWidth, 2)}%` }}></div>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>
              </div>
            )}

            {/* VISTA 8: APROBACIONES DE USUARIOS */}
            {activeTab === 'approvals' && (currentUser.role === 'ADMIN' || currentUser.role === 'MASTER') && (
              <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-4">
                <h2 className="text-lg font-bold text-white flex items-center gap-2">
                  <i data-lucide="user-check" className="text-amber-400"></i> Solicitudes de Registro Pendientes
                </h2>
                {pendingUsers.length === 0 ? (
                  <p className="text-slate-500 text-xs italic">No hay solicitudes pendientes.</p>
                ) : (
                  pendingUsers.map(u => (
                    <div key={u.code} className="p-4 bg-slate-950 border border-slate-800 rounded-xl flex justify-between items-center text-xs">
                      <div>
                        <span className="font-bold text-white">{u.firstName} {u.lastName}</span>
                        <span className="block text-slate-400">CÃ³digo: {u.code} | CÃ©dula: {u.idCard} | Rol Solicitado: {u.role}</span>
                      </div>
                      <button onClick={() => { setUsers([...users, { ...u, name: `${u.firstName} ${u.lastName}`, status: 'APPROVED' }]); setPendingUsers(pendingUsers.filter(x => x.code !== u.code)); }} className="bg-emerald-600 text-white font-bold px-4 py-2 rounded-xl">Aprobar Usuario</button>
                    </div>
                  ))
                )}
              </div>
            )}
          </main>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
  </script>
</body>
</html>
